# *de novo* variant calling workflow, CPU version
Maintainer: Jeffrey Ng

Work in progress!

Uses only CPU based programs to call de novo variants

Testing progress:

Docker -> builds

Workflow -> Finish running, fixed errors.

GPU comparison -> High levels of overlap with GPU output

Larger scale test -> Run on 3 more trios, with similar high levels of overlap

Resource/run time optimization -> GATK thread optimization -> Default seems to be the best balance of run time and CPU usage (no change in run time, change in CPU usage)

Add more user options for testing (regions not included), pathway checks for config file.

Need a local version!! -> working.

Finish readme plus others test -> ongoing

============ -> README START


### Developed and maintained by Jeffrey Ng
### Washington University in St. Louis Medical School
### Turner Lab

This pipeline takes in aligned, short read Illumina data, as cram or bam files, for a family trio and reports *de novo* variants in a vcf file.  This is a modified version pipeline run from [Ng et al. 2021](https://www.biorxiv.org/content/10.1101/2021.05.27.445979v1.full), making use of the open source, CPU versions of the programs that were GPU accelerated in the paper. It uses DeepVariant [(Poplin et al. 2018)](https://rdcu.be/7Dhl) and GATK Haplotypecaller to call variants [(Poplin et al. 2017)](https://www.biorxiv.org/content/10.1101/201178v3), GLnexus [(Lin et al. 2018)](https://www.biorxiv.org/content/10.1101/343970v1) for joint-genotyping , and lastly custom scripts to call *de novo* variants.  


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


Example .bam files are found in the test_code folder for testing purposes.


To download our GRCh38 reference, please use the following:
```
wget -q http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/GRCh38_reference_genome/GRCh38_full_analysis_set_plus_decoy_hla.fa
```


If you would like to download the RepeatMasker files, please use the following links:

#### Centromeres
```
wget -q https://de.cyverse.org/dl/d/B42A0F3D-C402-4D5F-BBD5-F0E61BE2F4AC/hg38_centromeres_09252018.bed.gz
wget -q https://de.cyverse.org/dl/d/37B13DB5-0478-4C4B-B18D-33AFB742E782/hg38_centromeres_09252018.bed.gz.tbi
```

#### LCR plus 5 bp buffer
```
wget -q https://de.cyverse.org/dl/d/870755FF-CD04-4010-A1EC-658D7E1151EF/LCR-hs38-5bp-buffer.bed.gz
wget -q https://de.cyverse.org/dl/d/01D038EA-51CC-4750-9814-0BB3784E808E/LCR-hs38-5bp-buffer.bed.gz.tbi
```

#### Recent repeats plus 5 bp buffer
```
wget -q https://de.cyverse.org/dl/d/185DA9BC-E13D-429B-94EA-632BDAB4F8ED/recent_repeat_b38-5bp-buffer.bed.gz
wget -q https://de.cyverse.org/dl/d/4A6AF6EF-D3F0-4339-9B8E-3E9E83638F00/recent_repeat_b38-5bp-buffer.bed.gz.tbi
```

#### CpG Locations
```
wget -q https://de.cyverse.org/dl/d/786D1640-3A26-4A1C-B96F-425065FBC6B7/CpG_sites_sorted_b38.bed.gz
wget -q https://de.cyverse.org/dl/d/713F020E-246B-4C47-BBC3-D4BB86BFB6E9/CpG_sites_sorted_b38.bed.gz.tbi
```

Before running, please make any necessary changes to these options below in the config.json. 

* regions:  "/region" *If you don't have the RepeatMasker files, please make this entry blank*
* gq_value:  20 *gq value filter*
* depth_value: 10 *Depth value filter*
* suffix: "\_test.bam" *Suffix of your data files.  Assumes input files are \<sample\_name\>\<suffix\>* 



## Running

All software dependencies are installed in a docker image that we provide for you at `tnturnerlab/dnv_wf_cpu`
We also provide the Dockerfile, if you would like to make modifications.  [Please see below](#docker-build-instructions) to find instructions on how the docker image was built.  

### Running on a LSF server

First, set up your LSF_DOCKER_VOLUMES:
```
export LSF_DOCKER_VOLUMES="/path/to/crams/:/data_dir /path/to/reference:/reference /path/to/this/git/repo/:/dnv_wf_cpu/ /path/to/RepeatMasker/files:/region"
```

Then run this command:
```
bsub -q general -oo %J.main.log -R 'span[hosts=1] rusage[mem=5GB]' -a 'docker(tnturnerlab/dnv_wf_cpu)' /miniconda3/bin/snakemake -j 6 --cluster-config cluster_config.json --cluster "bsub -q tychele -R 'span[hosts=1] rusage[mem={cluster.mem}]' -n {cluster.n} -M {cluster.mem} -a 'docker(tnturnerlab
/dnv_wf_cpu)' -M {cluster.mem} -oo log/%J.log.txt" -s gatk_deep_glnexus_qol.snake -k --rerun-incomplete -w 120 
```

### Running locally

This code can be run locally.  It will use all possible CPU cores, as well as whatever RAM is free at run time.

To run this locally, please run this command:
```
docker run -v "/path/to/crams/:/data_dir" -v "/path/to/reference:/reference" -v "/path/to/this/git/repo/:/dnv_wf_cpu/" -v "/path/to/RepeatMasker/files:/region" tnturnerlab/dnv_wf_cpu /miniconda3/bin/snakemake -s /dnv_wf_cpu/gatk_deep_glnexus_wf_local.snake -k --rerun-incomplete -w 120
```

## Output

### With the regions option

After the run, you'll find two main output files found in the folder called `out`:
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\_position.vcf
    * This file holds the *de novo* variants
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\_all.vcf
    * This file holds the *de novo* variants specifically within CpG regions.  The number of variants in this file should make 18%-20% of the total *de novo* variants found.

### Without the region option

Your main output file is:
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\.vcf
  * This file holds *de novo* variants that are not filtered by regions mentioned above.  The other two files will just contain the header file, but will be empty otherwise.


When running the small test bams found in this git repo, you should get these two DNVs in either NA12878.glnexus.family.combined_intersection_filtered_gq_20_depth_10.vcf or NA12878.glnexus.family.combined_intersection_filtered_gq_20_depth_10_position.vcf:

```
chr5	51158671	chr5_51158671_A_G	A	G	43	.	AC=1;AF=0.167;AN=6;INH=denovo_pro;TRANSMITTED=no;set=Intersection	GT:AD:DP:GQ:PL:RNC	0/1:20,10:30:44:43,0,58	0/0:26,0:26:50:0,123,1229	0/0:19,0:19:50:0,81,809
chr5	52352927	chr5_52352927_T_C	T	C	56	.	AC=1;AF=0.167;AN=6;INH=denovo_pro;TRANSMITTED=no;set=Intersection	GT:AD:DP:GQ:PL:RNC	0/1:17,15:32:55:56,0,61	0/0:28,0:28:50:0,102,1019	0/0:26,0:26:50:0,114,1139
```

## GPU vs CPU workflow output comparison

We have seen that our CPU pipeline shows high levels of overlap between the discovered number of *de novo* variants and can be used as an accurate alternative to the GPU accelerated version of the pipeline detailed in Ng et al. 2021.  

Here are two trios, NA12878 and NA12940, and the comparison of the number of *de novo* variants.

### NA12878

![NA12878 comparison](https://github.com/TNTurnerLab/dnv_workflow_cpu/blob/main/docs/GPU_vs_CPU_NA12878.png)

### NA19240

![NA19240 comparison](https://github.com/TNTurnerLab/dnv_workflow_cpu/blob/main/docs/GPU_vs_CPU_NA19240.png)

Run time information can be found within the docs folder.  Run times were run on a LSF based HCP 



## Software Requiments

* All software requirments are built in to the docker image.  The  software packages built are as follows:
  * DeepVariant v0.10.0
  * GATK v4.1.0.0
  * GLnexus v1.2.6
  * Snakemake v3.12.0
  * python 3.6.5
  * python 2.7
  * BCFtools v1.9
  * SAMtools v1.9
  * bedtools v2.29.2
  * tabix 
  * vcflib
  * openjdk 8

## Docker build instructions
  
* If you  want to build the docker image from the Dockerfile, you'll need to be inside the DeepVariant code base (we build DeepVariant from v0.10.0 source).  
```
wget https://github.com/google/deepvariant/archive/refs/tags/v0.10.0.zip
unzip v0.10.0.zip
```

* You can then copy our dockerfile into this folder
```
cp dnv_workflow_cpu/dockerfiles/Dockerfile deepvariant-0.10.0
``` 

* Lastly, go inside the DeepVariant folder and change the settings.sh file and change the export DV_USE_GCP_OPTIMIZED_TF_WHL to 0, this can be found on line 91.

```
export DV_USE_GCP_OPTIMIZED_TF_WHL="${DV_USE_GCP_OPTIMIZED_TF_WHL:-0}"
```
* Now you can build the docker image from inside the DeepVariant folder.


```
docker build deepvariant-0.10.0/
```
