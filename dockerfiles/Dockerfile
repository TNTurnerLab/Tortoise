FROM continuumio/miniconda3:latest
RUN apt-get update && apt-get install -y --no-install-recommends wget unzip bzip2 git libnss-sss && rm -rf /var/lib/apt/lists* \
    && conda install python=3.6.5 \
    && conda install -c bioconda bcftools=1.9\
    && conda install -c bioconda tabix \
    && conda install -c bioconda vcflib \
    && conda install -c bioconda bedtools=2.29.2 \
    && conda install -c bioconda snakemake \
    && conda install -c bioconda openjdk=8 \
    && conda install -c bioconda samtools=1.9 
   
RUN conda install -c bioconda gatk4=4.1.0.0
    
RUN  wget https://storage.googleapis.com/gatk-software/package-archive/gatk/GenomeAnalysisTK-3.5-0-g36282e4.tar.bz2 \
    &&  tar -xjf GenomeAnalysisTK-3.5-0-g36282e4.tar.bz2
    
RUN wget https://github.com/dnanexus-rnd/GLnexus/releases/download/v1.2.6/glnexus_cli \ 
&& chmod +x glnexus_cli

   
ENV PATH "PATH=/miniconda/bin:${PATH}"
RUN wget  https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh \
&& bash Miniconda2-latest-Linux-x86_64.sh -p /miniconda -b
RUN wget https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-337.0.0-linux-x86_64.tar.gz \
&& tar -xf google-cloud-sdk-337.0.0-linux-x86_64.tar.gz \
&& /google-cloud-sdk/install.sh

FROM ubuntu:16.04



RUN apt-get update && apt-get install --no-install-recommends -y apt-transport-https curl gnupg  wget ca-certificates  libnss-sss git unzip gzip && rm -rf /var/lib/apt/lists* 


RUN curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor > bazel.gpg \
&&  mv bazel.gpg /etc/apt/trusted.gpg.d/ \ 
&& echo "deb [arch=amd64] https://storage.googleapis.com/bazel-apt stable jdk1.8" |  tee /etc/apt/sources.list.d/bazel.list \
&&  apt update &&  apt install -y bazel \
&&  apt update &&  apt full-upgrade -y
RUN git clone https://github.com/google/deepvariant.git && PATH="${HOME}/bin:${PATH}" \
&& cd deepvariant \
&& ./build-prereq.sh \
&& ./build_and_test.sh
RUN cd deepvariant \
&& ./run-prereq.sh

#RUN wget https://github.com/google/deepvariant/releases/download/v0.10.0/deepvariant.zip --no-check-certificate  
#RUN git clone https://github.com/google/deepvariant.git && PATH="${HOME}/bin:${PATH}" \
#&& cd deepvariant \
#&& ./build_release_binaries.sh /deepvariant.zip


CMD ["/bin/bash"]
