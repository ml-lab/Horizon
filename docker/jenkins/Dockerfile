# Pre-req installations:
# https://docs.docker.com/install/linux/docker-ce/ubuntu/
# https://github.com/NVIDIA/nvidia-docker

# Usage:
#        sudo docker build -t horizon_initial_release . 2>&1 | tee stdout
#    or
#        sudo nvidia-docker build -t horizon_initial_release . 2>&1 | tee stdout
#        sudo nvidia-docker run -i -t --rm horizon_initial_release /bin/bash

# Remove all stopped Docker containers:   sudo docker rm $(sudo docker ps -a -q)
# Remove all untagged images:             sudo docker rmi $(sudo docker images -q --filter "dangling=true")

# Available versions: https://hub.docker.com/r/nvidia/cuda/
FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04

SHELL ["/bin/bash", "-c"]

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    cmake \
    git \
    libgflags-dev \
    libgoogle-glog-dev \
    libgtest-dev \
    libiomp-dev \
    libleveldb-dev \
    liblmdb-dev \
    libopencv-dev \
    libopenmpi-dev \
    libprotobuf-dev \
    libsnappy-dev \
    locales \
    openmpi-bin \
    openmpi-doc \
    protobuf-compiler \
    sudo \
    software-properties-common \
    wget

# Sometimes needed to avoid SSL CA issues.
RUN update-ca-certificates

# Need this stuffs to build thrif compiler
RUN apt-get install -y --no-install-recommends \
    automake \
    bison \
    flex \
    g++ \
    libboost-all-dev \
    libevent-dev \
    libssl-dev \
    libtool \
    make \
    pkg-config

RUN wget https://archive.apache.org/dist/thrift/0.11.0/thrift-0.11.0.tar.gz -O thrift-0.11.0.tar.gz && \
    tar -xzf thrift-0.11.0.tar.gz && \
    cd thrift-0.11.0 && \
    ./bootstrap.sh && \
    ./configure --without-java && \
    make && \
    make install

# Install Java & maven.
# Taken from https://github.com/dockerfile/java/blob/master/oracle-java8/Dockerfile
RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer maven && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

ENV HOME /home
WORKDIR ${HOME}/

RUN wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh && \
    chmod +x miniconda.sh && \
    ./miniconda.sh -b -p ${HOME}/miniconda && \
    rm miniconda.sh

# Setting these env var outside of the install script to ensure
# they persist in image
# (See https://stackoverflow.com/questions/33379393/docker-env-vs-run-export)
ENV PATH ${HOME}/miniconda/bin:$PATH
ENV CONDA_PATH ${HOME}/miniconda

# Needed to prevent UnicodeDecodeError: 'ascii' codec can't decode byte
# when installing fairseq.
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Reminder: this should be updated when switching between CUDA 8 or 9. Should
# be kept in sync with TMP_CUDA_VERSION in install_prereqs.sh
ENV NCCL_ROOT_DIR ${HOME}/horizon/nccl_2.1.15-1+cuda9.0_x86_64
ENV LD_LIBRARY_PATH ${CONDA_PATH}/lib:${NCCL_ROOT_DIR}/lib:${LD_LIBRARY_PATH}

ADD ./install_prereqs.sh install_prereqs.sh
ADD ./requirements.txt requirements.txt
ARG USE_GPU
RUN ./install_prereqs.sh ${USE_GPU}
RUN rm install_prereqs.sh requirements.txt

# Add Jenkins user
ARG JENKINS
ARG JENKINS_UID
ARG JENKINS_GID
ADD ./add_jenkins_user.sh add_jenkins_user.sh
RUN if [ -n "${JENKINS}" ]; then bash ./add_jenkins_user.sh ${JENKINS_UID} ${JENKINS_GID}; fi
RUN rm add_jenkins_user.sh

# Define default command.
CMD ["bash"]