FROM jenkins/ssh-agent:debian-jdk21

ENV HOME /home/jenkins-slave
USER root

# install netstat to allow connection health check with
# netstat -tan | grep ESTABLISHED
RUN apt-get update && apt-get install -y net-tools && rm -rf /var/lib/apt/lists/*

RUN useradd -c "Jenkins Slave user" -d $HOME -m jenkins-slave

# Install Docker
RUN apt-get update \
  && apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common net-tools wget git build-essential
RUN curl -fsSL https://get.docker.com -o get-docker.sh
RUN sh get-docker.sh
RUN apt-get update && apt-get install -y maven autoconf automake libtool curl make g++ unzip 

# ensure local python is preferred over distribution python
ENV PATH /usr/local/bin:$PATH

# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

# extra dependencies (over what buildpack-deps already includes)
RUN apt-get update && apt-get install -y --no-install-recommends \
        tk-dev libssl-dev g++ apt-transport-https ca-certificates gnupg zip libsqlite3-dev wget gcc make zlib1g-dev libbz2-dev liblzma-dev libffi-dev \
    && rm -rf /var/lib/apt/lists/*

ARG BUILDDIR="/tmp/build"
ARG PYTHON_VER="3.8.6"
WORKDIR ${BUILDDIR}

RUN wget --quiet https://www.python.org/ftp/python/${PYTHON_VER}/Python-${PYTHON_VER}.tgz > /dev/null 2>&1 && \
  tar zxf Python-${PYTHON_VER}.tgz && \
  cd Python-${PYTHON_VER} && \
  ./configure --enable-loadable-sqlite-extensions > /dev/null 2>&1 && \
  make > /dev/null 2>&1 && \
  make install > /dev/null 2>&1 && \
  rm -rf ${BUILDDIR} 

# make some useful symlinks that are expected to exist
RUN cd /usr/local/bin \
    && ln -s idle3 idle \
    && ln -s pydoc3 pydoc \
    && ln -s python3 python \
    && ln -s python3-config python-config

# Install pip library
RUN apt-get update && apt-get install -y python3-pip

# Install helm
RUN wget https://get.helm.sh/helm-v2.15.1-linux-amd64.tar.gz && \
  tar -xzf helm-v2.15.1-linux-amd64.tar.gz && \
  install linux-amd64/helm /usr/local/bin/

ENTRYPOINT ["jenkins-slave"]