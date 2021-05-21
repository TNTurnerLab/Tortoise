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
7b60651143aa1b18e807ced767886fd2


============

# How to Run

## Input

Two main input:

1) .cram files for trio(s)
2) A comma delimited text file filled with trios in the following order:  Father, Mother, Child

To download 30x WGS NA12878 trio .cram files from the 1000 Genomes Project to test the workflow, you can find the download links below:

```
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR323/ERR3239334/NA12878.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR323/ERR3239334/NA12878.final.cram.crai
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989341/NA12891.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989341/NA12891.final.cram.crai
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989342/NA12892.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989342/NA12892.final.cram.crai
```

## Running

Before running, please change the config file so that it will point to all of the correct directories.  

* reference:  Reference file
* family_file:  Trio family file, comma delimited
* gatk_out:  Pathway to where the GATK output will go
* dv_out:  Pathway to where the DeepVaraint output will go
* glnexus_file_dv_bcf:  Pathway to where the GLnexus .bcf file will be generated for DeepVariant gvcfs
* glnexus_file_hc_bcf:  Pathway to where the GLnexus .bcf file will be generated for GATK gvcfs
* glnexus_file_dv_vcf:  Pathway to where the joint genotyped .vcf file will be generated from DeepVariant gvcfs
* glnexus_file_hc_vcf:  Pathway to where the joint genotyped .vcf file will be generated from GATK gvcfs
* glnexus_file_dv:  Pathway to where the vcf output from GLnexus run on DeepVariant is.  
* glnexus_file_hc:  Pathway to where the vcf output from GLnexus run on GATK Haploytpecaller is.  
* regions:  Folder to where RepeatMaster files are for region based filtering.
* gq_value:  gq value filter.
* depth_value: Depth value filter.
* out_dir:  Output directory.  

To run the snake file generally, it can run by this:
```
snakemake -s dnv_wf_cpu_test.snake  -k --rerun-incomplete -w 120
```


To run the snake on a server cluster, specifically one running LSF, it can be run like this:

```
bsub -g general -M 5G -oo %J.main.log -R 'span[hosts=1] rsuage[mem=5G]' -n 1 -a 'docker(jng2/testrepo2_actual:dn_wf_cpu_deep.0.1)' /miniconda3/bin/snakemake -s dnv_wf_cpu_test.snake --cluster-config cluster_config.json -j 6 --cluster "bsub -g general -M {cluster.mem} -n {cluster.n}  -R 'span[hosts=1] rusage[mem={cluster.mem}]'  -oo log/%J.log.txt"  -k --rerun-incomplete -w 120
```



