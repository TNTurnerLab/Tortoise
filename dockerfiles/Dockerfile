###
# Created by Jeffrey Ng, Turner Lab WashU
# Note DeepVariant build instructions are taken from v.1.4 of Google's DeepVariant page 
###


FROM continuumio/miniconda3:4.12.0 as conda_setup 
RUN conda install -c bioconda bcftools=1.11 \
    && conda install -c bioconda tabix=1.11 \
    && conda install -c bioconda vcflib=1.0.0-rc0 \
    && conda install -c bioconda bedtools=2.29.2 \
    && conda install -c bioconda samtools=1.11 \
    && conda install -c bioconda gatk4=4.2.0.0
RUN conda create -n snake -c bioconda -c conda-forge snakemake=7.15.2-0
RUN conda create -n py2 python=2.7

FROM ubuntu:20.04
RUN apt-get update && apt-get install --no-install-recommends -y ca-certificates apt-transport-https libnss-sss git gcc g++ vim less unzip wget tabix && rm -rf /var/lib/apt/lists*

COPY --from=conda_setup /opt/conda /opt/conda

RUN cd /opt && git clone https://github.com/google/deepvariant.git 



WORKDIR /opt/deepvariant
RUN echo "Acquire::http::proxy \"$http_proxy\";\n" \
         "Acquire::https::proxy \"$https_proxy\";" > "/etc/apt/apt.conf"

RUN ./build-prereq.sh \
  && PATH="${HOME}/bin:${PATH}" ./build_release_binaries.sh  # PATH for bazel

WORKDIR /opt/
RUN cp /opt/deepvariant/bazel-bin/licenses.zip ../

WORKDIR /opt/deepvariant/bin/
RUN cp /opt/deepvariant/run-prereq.sh .
RUN cp /opt/deepvariant/settings.sh .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/make_examples.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/call_variants.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/call_variants_keras.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/postprocess_variants.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/vcf_stats_report.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/show_examples.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/runtime_by_region_vis.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/multisample_make_examples.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/model_train.zip .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/model_eval.zip  .
RUN cp /opt/deepvariant/bazel-out/k8-opt/bin/deepvariant/freeze_graph.zip  .
RUN cp /opt/deepvariant/scripts/run_deepvariant.py .

RUN ./run-prereq.sh
ARG PYTHON_VERSION=3.8
ARG VERSION=1.4.0
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 0 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python${PYTHON_VERSION} 0
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PYTHON_VERSION} 0 && \
    update-alternatives --install /usr/bin/python python /usr/bin/python${PYTHON_VERSION} 0

# Create shell wrappers for python zip files for easier use.
RUN \
  BASH_HEADER='#!/bin/bash' && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/make_examples.zip "$@"' > \
    /opt/deepvariant/bin/make_examples && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/call_variants.zip "$@"' > \
    /opt/deepvariant/bin/call_variants && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/call_variants_keras.zip "$@"' > \
    /opt/deepvariant/bin/call_variants_keras && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/postprocess_variants.zip "$@"' > \
    /opt/deepvariant/bin/postprocess_variants && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/model_train.zip "$@"' > \
    /opt/deepvariant/bin/model_train && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/model_eval.zip "$@"' > \
    /opt/deepvariant/bin/model_eval && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/vcf_stats_report.zip "$@"' > \
    /opt/deepvariant/bin/vcf_stats_report && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/show_examples.zip "$@"' > \
    /opt/deepvariant/bin/show_examples && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/runtime_by_region_vis.zip "$@"' > \
    /opt/deepvariant/bin/runtime_by_region_vis && \
    printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/multisample_make_examples.zip "$@"' > \
    /opt/deepvariant/bin/multisample_make_examples && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 /opt/deepvariant/bin/freeze_graph.zip "$@"' > \
    /opt/deepvariant/bin/freeze_graph && \
  printf "%s\n%s\n" \
    "${BASH_HEADER}" \
    'python3 -u /opt/deepvariant/bin/run_deepvariant.py "$@"' > \
    /opt/deepvariant/bin/run_deepvariant && \
  chmod +x /opt/deepvariant/bin/make_examples \
    /opt/deepvariant/bin/call_variants \
    /opt/deepvariant/bin/call_variants_keras \
    /opt/deepvariant/bin/postprocess_variants \
    /opt/deepvariant/bin/vcf_stats_report \
    /opt/deepvariant/bin/show_examples \
    /opt/deepvariant/bin/runtime_by_region_vis \
    /opt/deepvariant/bin/multisample_make_examples \
    /opt/deepvariant/bin/model_train \
    /opt/deepvariant/bin/model_eval \
    /opt/deepvariant/bin/run_deepvariant \
    /opt/deepvariant/bin/freeze_graph

# Copy models
WORKDIR /opt/models/wgs
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wgs_standard/model.ckpt.data-00000-of-00001 .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wgs_standard/model.ckpt.index .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wgs_standard/model.ckpt.meta .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wgs_standard/model.ckpt.example_info.json .
RUN chmod +r /opt/models/wgs/model.ckpt*

WORKDIR /opt/models/wes
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wes_standard/model.ckpt.data-00000-of-00001 .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wes_standard/model.ckpt.index .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wes_standard/model.ckpt.meta .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-wes_standard/model.ckpt.example_info.json .
RUN chmod +r /opt/models/wes/model.ckpt*

WORKDIR /opt/models/pacbio
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-pacbio_standard/model.ckpt.data-00000-of-00001 .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-pacbio_standard/model.ckpt.index .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-pacbio_standard/model.ckpt.meta .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-pacbio_standard/model.ckpt.example_info.json .
RUN chmod +r /opt/models/pacbio/model.ckpt*

WORKDIR /opt/models/hybrid_pacbio_illumina
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-hybrid_standard/model.ckpt.data-00000-of-00001 .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-hybrid_standard/model.ckpt.index .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-hybrid_standard/model.ckpt.meta .
ADD https://storage.googleapis.com/deepvariant/models/DeepVariant/${VERSION}/DeepVariant-inception_v3-${VERSION}+data-hybrid_standard/model.ckpt.example_info.json .
RUN chmod +r /opt/models/hybrid_pacbio_illumina/model.ckpt*



RUN apt-get -y update && \
  apt-get install -y parallel python3-pip && \
  PATH="${HOME}/.local/bin:$PATH" python3 -m pip install absl-py==0.13.0 && \
  apt-get clean autoclean && \
  apt-get autoremove -y --purge && \
  rm -rf /var/lib/apt/lists/*
WORKDIR /
RUN wget https://github.com/dnanexus-rnd/GLnexus/releases/download/v1.4.1/glnexus_cli && chmod +x glnexus_cli
RUN apt-get update && apt-get install -y --no-install-recommends gcc libz-dev libopenblas-base && pip3 install pytabix

CMD ["/bin/bash"]

