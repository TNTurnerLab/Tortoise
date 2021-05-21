# dnv_workflow_cpu

Maintainer:  Jeffrey Ng

Work in progress!

Uses only CPU based programs to call de novo variants

Testing progress:

Docker -> builds

Workflow -> Finish running, fixed errors. 

GPU comparison -> High levels of overlap with GPU output

Larger scale test -> Run on 3 more trios, with similar high levels of overlap

Resource/run time optimization -> GATK thread optimization -> Default seems to be the best balance of run time and CPU usage (no change in run time, change in CPU usage)

Add more user options for testing (regions not included), pathway checks for config file.

Need a local version!!

============ -> README START
# *de novo* variant calling workflow

This pipeline takes in aligned, short read Illumina data, as cram files, for a family trio and reports *de novo* variants in a vcf file.  This is a modified version pipeline run from Ng et al. 2021 (hopefully! :D), making use of the open source, CPU versions of the programs that were GPU accelerated in the paper. 


# How to Run

## Input

Two main input:

1) .cram files for trio(s)
2) A comma delimited text file filled with trios in the following order:  Father, Mother, Child
3) The snake config file

To download 30x WGS NA12878 trio .cram files from the 1000 Genomes Project to test the workflow, you can find the download links below:

```
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR323/ERR3239334/NA12878.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR323/ERR3239334/NA12878.final.cram.crai
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989341/NA12891.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989341/NA12891.final.cram.crai
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989342/NA12892.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989342/NA12892.final.cram.crai
```

An example family file for this trio is provided in the test folder * **Make a test folder** *

Before running, please change the config file so that it will point to all of the correct directories.  

* reference:  Reference file
* family_file:  Trio family file, comma delimited
* gatk_out:  Pathway to where the GATK output will go
* dv_out:  Pathway to where the DeepVaraint output will go
* glnexus_file_dv_bcf:  Pathway to where the GLnexus .bcf file will be generated for DeepVariant gvcfs
* glnexus_file_hc_bcf:  Pathway to where the GLnexus .bcf file will be generated for GATK gvcfs
* glnexus_file_dv_vcf:  Pathway to where the joint genotyped .vcf file will be generated from DeepVariant gvcfs
* glnexus_file_hc_vcf:  Pathway to where the joint genotyped .vcf file will be generated from GATK gvcfs
* glnexus_file_dv:  Pathway to where the vcf output from GLnexus run on DeepVariant is 
* glnexus_file_hc:  Pathway to where the vcf output from GLnexus run on GATK Haploytpecaller is  
* regions:  Folder to where RepeatMasker files are for region based filtering 
* gq_value:  gq value filter
* depth_value: Depth value filter
* out_dir:  Output directory 

**Note for RepeatMasker files**

We filter *de novo* mutations by regions using files obtained from RepeatMasker.  We filter low complexity regions, recent repeats, centromeme regions, and CpG sites.  If you would like to use this option, you'll need to download the files yourself.  The link can be found here * **Find the link** *  If not, please keep the regions config blank. * **Implement blank region command** *.  

## Running

To run the snake on a LSF server cluster it can be run like this:

```
bsub -g general -M 5G -oo %J.main.log -R 'span[hosts=1] rsuage[mem=5G]' -n 1 -a 'docker(jng2/testrepo2_actual:dn_wf_cpu_deep.0.1)' /miniconda3/bin/snakemake -s dnv_wf_cpu_test.snake --cluster-config cluster_config.json -j 6 --cluster "bsub -g general -M {cluster.mem} -n {cluster.n}  -R 'span[hosts=1] rusage[mem={cluster.mem}]'  -oo log/%J.log.txt"  -k --rerun-incomplete -w 120
```

## Output

### With the regions option

After the run, you'll find two main output files:
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\_position.vcf
    * This file holds the *de novo* variants
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\_all.vcf
    * This file holds the *de novo* variants specifically within CpG regions.  The number of variants in this file should make 18%-20% of the total *de novo* variants found.

### Without the region option

You will find 1 main output file:
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\.vcf
  * This file hodl *de novo* variants that are not filtered by regions mentioned above.

## GPU vs CPU output comparison

We have seen that our CPU pipeline shows high levels of overlap between the discovered number of *de novo* variants and can be used as an accurate alternative to the GPU accelerated version of the pipeline.  

Here are two trios,NA12878 and NA12940, and the comparison of the number of *de novo* variants.

### NA12878

![NA12878 comparison](https://github.com/TNTurnerLab/dnv_workflow_cpu/blob/main/docs/GPU_vs_CPU_NA12878.png)

### NA19240

![NA19240 comparison](https://github.com/TNTurnerLab/dnv_workflow_cpu/blob/main/docs/GPU_vs_CPU_NA19240.png)

Run time information can be found within the docs folder.

## Software Requiments

* All software requirments are built in to the docker image.  The  software packages built are as follows:
  * DeepVariant v0.10.0
  * GATK v4.1.0.0
  * GLnexus v1.2.6
  * Snakemake v.3. **something**
  * python 3.6.5
  * python 2.7
  * BCFtools v1.9
  * SAMtools v1.9
  * bedtools v2.29.2
  * tabix 
  * vcflib
  * openjdk 8
  
* If you  want to build the docker image from the Dockerfile, you'll need to be inside the DeepVariant code base (we build DeepVariant from source).  The download link can be found [here.](https://github.com/google/deepvariant/archive/refs/tags/v0.10.0.zip)
* You can then copy our dockerfile into the folder and change the settings.sh file and change the export DV_USE_GCP_OPTIMIZED_TF_WHL to 0

```
export DV_USE_GCP_OPTIMIZED_TF_WHL="${DV_USE_GCP_OPTIMIZED_TF_WHL:-0}"
```

