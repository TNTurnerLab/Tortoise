# Tortoise:  *de novo* variant calling workflow, CPU version
 
### Developed and maintained by Jeffrey Ng
### Washington University in St. Louis Medical School
### Tychele N. Turner, Ph.D., Lab
 
This pipeline takes in aligned, short read Illumina data, in the form of .cram or .bam files, for a family trio and reports *de novo* variants in a .vcf file.  This is a modified version pipeline run from [Ng et al. 2022](https://doi.org/10.1002/humu.24455), making use of the open source, CPU only versions of the programs that were GPU accelerated in the paper. This pipeline uses DeepVariant [(Poplin et al. 2018)](https://rdcu.be/7Dhl) and GATK Haplotypecaller to call variants [(Poplin et al. 2017)](https://www.biorxiv.org/content/10.1101/201178v3), GLnexus [(Lin et al. 2018)](https://www.biorxiv.org/content/10.1101/343970v1) for joint-genotyping , and lastly, a custom workflow to call *de novo* variants.  
 
# How to Run
 
## Input
 
Two main inputs:
 
1) .bam or .cram files for the trio(s)
2) A comma delimited text file, with one trio per line, with sample IDs formatted in the following way:  Father,Mother,Child
 
You may download the full 30x WGS NA12878 trio .cram files from the 1000 Genomes Project to test the workflow, links found below:
 
```
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR323/ERR3239334/NA12878.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR323/ERR3239334/NA12878.final.cram.crai
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989341/NA12891.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989341/NA12891.final.cram.crai
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989342/NA12892.final.cram
wget -q ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR398/ERR3989342/NA12892.final.cram.crai
```
 
Alternatively, small example .bam files can be found in the `test_code` folder for faster testing purposes.  These .bam files were created from the 30x .cram files above, isolating a small region on chromosome 5.  
 
To download the GRCh38 reference, please use the following:
```
wget -q http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/GRCh38_reference_genome/GRCh38_full_analysis_set_plus_decoy_hla.fa
wget -q http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/GRCh38_reference_genome/GRCh38_full_analysis_set_plus_decoy_hla.fa.fai
wget -q http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/GRCh38_reference_genome/GRCh38_full_analysis_set_plus_decoy_hla.dict
 
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
 


## Running
 
This pipeline can be run using the Docker image found here: `tnturnerlab/tortoise:v1.1`
We also provide the Dockerfile if you would like to make modifications.  

### Snakemake

#### Setting up the config.json file

Before running, please make any necessary changes to these options below in the config.json. 
 
* regions:  "/region" *If you don't have the RepeatMasker files, please make this entry blank*
* gq_value:  20 *gq value filter*
* depth_value: 10 *Depth value filter*
* suffix: "\_supersmall.bam" *Suffix of your data files.  Assumes input files are \<sample\_name\>\<suffix\>* 
* family_file: "/dnv_wf_cpu/<your_family_file>"
* chrom_length: Optional chromosome length file, use if you are not using human reference GRCh38. Can leave blank if using GRCh38. Please make this a two column, tab delimited file, with the first chromosome and the second column the length of the chromosome



### Running on a LSF server
 
First, set up the LSF_DOCKER_VOLUMES:
```
export LSF_DOCKER_VOLUMES="/path/to/crams/:/data_dir /path/to/reference:/reference /path/to/this/git/repo/:/dnv_wf_cpu/ /path/to/RepeatMasker/files:/region"
```
 
Then run this command:
```
bsub -q general -oo %J.main.log -R 'span[hosts=1] rusage[mem=5GB]' -a 'docker(tnturnerlab/tortoise:v1.1)' /opt/conda/envs/snake/bin/snakemake -s /dnv_wf_cpu/tortoise_1.1.smk -j 6 --cluster-config cluster_config.json --cluster "bsub -q general -R 'span[hosts=1] rusage[mem={cluster.mem}]' -n {cluster.n} -M {cluster.mem} -a 'docker(tnturnerlab/tortoise:v1.1)' -M {cluster.mem} -oo %J.log.txt" -k --rerun-incomplete -w 120 
```
 
### Running locally
 
To run the small test samples provided, please ensure you have at least 16GB of RAM. To run a full sized trio, please ensure your machine has at least 64GB of RAM.  The pipeline will use as many CPUs as possible.  
 
To run this locally, please run this command:
 
```
docker run -v "/path/to/crams/:/data_dir" -v "/path/to/hare/code:/dnv_wf_cpu" -v "/path/to/reference:/reference"  -v "/path/to/RepeatMasker/region/files:/region" tnturnerlab/tortoise:v1.1 /opt/conda/envs/snake/bin/snakemake -s /dnv_wf_cpu/tortoise_1.1.smk -j 6 --cores -k --rerun-incomplete -w 120 

```
 
#### Cromwell workflow

We also provide this workflow in a .wdl format.  Unlike the Snakemake, you will be able to run Parabricks directly from this workflow, instead of separately.  You can also run this workflow in the cloud.  To run this, you'll need to download the [Cromwell .jar found here](https://github.com/broadinstitute/cromwell/releases).  This wdl was specificlly tested on cromwell-83.  

The basic config file looks like this:
```
{
  "jumping_tortoise.glnexus_cpu": "Int (optional, default = 32)",
  "jumping_tortoise.glnexus_ram_hc": "Int (optional, default = 250)",
  "jumping_tortoise.maxPreemptAttempts": "Int (optional, default = 3)",
  "jumping_tortoise.extra_mem_hc": "Int (optional, default = 65)",
  "jumping_tortoise.test_intersect": "File", #pathway to the test_intersect.py file
  "jumping_tortoise.chrom_length": "File? (optional)",  #Optional chromosome length file if you are not using Human build GRCh38
  "jumping_tortoise.combinedAndFilter.extramem_GLDV": "Int? (optional)",
  "jumping_tortoise.pathToReference": "File",  #pathway to tarball of reference information
  "jumping_tortoise.glnexus_ram_dv": "Int (optional, default = 100)",
  "jumping_tortoise.wes": "Boolean (optional, default = false)",   #Please set this to true if you are analyzing WES data
  "jumping_tortoise.glnexus_DV.extramem_GLDV": "Int? (optional)",
  "jumping_tortoise.hare_docker": "String (optional, default = \"tnturnerlab/hare:v1.1\")",
  "jumping_tortoise.cram_files": "Array[Array[WomCompositeType {\n cram -> File\ncrai -> File \n}]]",  #cram/bam file input, please see example for formating
  "jumping_tortoise.regions": "File? (optional)",
  "jumping_tortoise.cpu_dv": "Int (optional, default = 32)",
  "jumping_tortoise.naive_inheritance_trio_py2": "File",
  "jumping_tortoise.sample_suffix": "String",  #suffix of the input cram file.  If your sample was NA12878.final.cram, you would put ".final.cram" here
  "jumping_tortoise.num_ram_dv": "Int (optional, default = 120)",
  "jumping_tortoise.glnexus_HC.extramem_GLDV": "Int? (optional)",
  "jumping_tortoise.deep_docker": "String (optional, default = \"tnturnerlab/tortoise:v1.1\")",
  "jumping_tortoise.filter_glnexuscombined_updated": "File",
  "jumping_tortoise.gq": "Int (optional, default = 20)",
  "jumping_tortoise.extra_mem_dv": "Int (optional, default = 65)",
  "jumping_tortoise.trios": "Array[WomCompositeType {\n father -> String\nmother -> String\nchild -> String \n}]",   #trios MUST be in same order as trios in cram_file
  "jumping_tortoise.depth": "Int (optional, default = 10)",
  "jumping_tortoise.deep_model": "String (optional, default = \"WGS\")",
  "jumping_tortoise.num_ram_hc": "Int (optional, default = 64)",
  "jumping_tortoise.interval_file": "String (optional, default = \"None\")",
  "jumping_tortoise.glnexus_deep_model": "String (optional, default = \"DeepVariant\")",
  "jumping_tortoise.reference": "String",
  "jumping_tortoise.cpu_hc": "Int (optional, default = 4)"
} 
```
Required arguments are highlighted in comments above.  We have provided an example config to help with formatting. Please modify the computational requirements to fit your HPC.  If you are running it on Google CLoud Platform, you may keep the computation settings. Requirements are based on [NVIDIA's own workflows found here.](https://github.com/clara-parabricks-workflows/parabricks-wdl)  If you are going to use this wdl, please tarball your reference files.  If you are running WES data, please include your capture region in this tarball.
```
tar -jcf reference.tar.bz2 reference.fa reference.fa.fai reference.dict
```


## Output
 
### With the regions option
 
After the run, you'll find two main output files found in the folder called `out`:
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\_position.vcf
    * This file holds the *de novo* variants
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\_position_all.vcf
    * This file holds the *de novo* variants specifically within CpG regions.  
 
### Without the region option
 
Your main output file is:
* <child_name>.glnexus.family.combined_intersection_filtered_gq_<gq_value>\_depth_<depth_value>\.vcf
  * This file holds *de novo* variants that are not filtered by regions mentioned above.  The other two files will just contain the header but otherwise will be empty.
 
When running the small test bams, you will find these two DNVs in both `NA12878.glnexus.family.combined_intersection_filtered_gq_20_depth_10.vcf` or `NA12878.glnexus.family.combined_intersection_filtered_gq_20_depth_10_position.vcf`:
 
```
chr5 	51158671    	chr5_51158671_A_G A      	G     	43    	.        	AC=1;AF=0.167;AN=6;INH=denovo_pro;TRANSMITTED=no;set=Intersection  	GT:AD:DP:GQ:PL:RNC     	0/1:20,10:30:44:43,0,58        	0/0:26,0:26:50:0,123,1229 	0/0:19,0:19:50:0,81,809
chr5 	52352927    	chr5_52352927_T_C  T      	C     	56    	.        	AC=1;AF=0.167;AN=6;INH=denovo_pro;TRANSMITTED=no;set=Intersection  	GT:AD:DP:GQ:PL:RNC     	0/1:17,15:32:55:56,0,61        	0/0:28,0:28:50:0,102,1019 	0/0:26,0:26:50:0,114,1139
```

## Software Requirements
 
* All software requirements are built in to the docker image.  The  software packages built are as follows:
  * DeepVariant v1.4
  * GATK v4.2.0.0
  * GLnexus v1.4.1
  * Snakemake v7.15.2-0
  * python 3.9.7
  * python 2.7
  * BCFtools v1.11
  * SAMtools v1.11
  * bedtools v2.29.2
  * tabix v1.11
  * vcflib v1.0.0-rc0
 

 


