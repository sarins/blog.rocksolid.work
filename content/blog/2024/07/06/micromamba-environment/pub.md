+++
title = 'Micromamba环境准备'
subtitle = 'Micromamba Environment Preparation'
brief = ''
date = 2024-07-06T00:00:00
categories = ['Python', 'Micromamba', 'Jupyter', 'Conda']
series = ['Python Language']
tags = ['Python', 'Micromamba', 'Jupyter', 'Conda', 'Jupyterlab']
type = 'blog'
+++

---

## 1. Get image

> Get images from [**QuantStack MambaOrg Image repository**](https://hub.docker.com/r/mambaorg/micromamba/tags)

> [**Micromamba docker document**](https://micromamba-docker.readthedocs.io/en/latest/index.html)

```bash
# Chosen version 1.5.8-jammy-cuda-11.8.0 at this time.
docker pull mambaorg/micromamba:1.5.8-jammy-cuda-11.8.0
```

---

## 2. Importants

> Running commands in Dockerfile within the conda environment
> 1. Set ARG MAMBA_DOCKERFILE_ACTIVATE=1 to activate the conda environment
> 2. Use the ‘shell’ form of the RUN command
> 3. More detail will be found in [**Running commands in Dockerfile within the conda environment**](https://micromamba-docker.readthedocs.io/en/latest/quick_start.html#running-commands-in-dockerfile-within-the-conda-environment)
> 4. About how to making smaller images will be found in [**Deploying conda environments in (Docker) containers - how to do it right**](https://uwekorn.com/2021/03/01/deploying-conda-environments-in-docker-how-to-do-it-right.html)

---

## 2. Create customized image based on Micromamba image

### 2.1. Dockerfile

```dockerfile
# Base micromamba image
FROM mambaorg/micromamba:1.5.8-jammy-cuda-11.8.0

ARG ROCKSOLID_USER=rocksolid
ARG ROCKSOLID_UID=1000
ARG ROCKSOLID_GID=100

USER root

RUN usermod "--login=${ROCKSOLID_USER}" "--home=/home/${ROCKSOLID_USER}" --move-home "-u ${ROCKSOLID_UID}" "${MAMBA_USER}" && \
    groupmod "--new-name=${ROCKSOLID_USER}" --non-unique "-g ${ROCKSOLID_GID}" "${MAMBA_USER}" && \
    # Update the expected value of MAMBA_USER for the
    # _entrypoint.sh consistency check.
    echo "${ROCKSOLID_USER}" > "/etc/arg_mamba_user" && \
    :
ENV MAMBA_USER=$ROCKSOLID_USER
ENV USER=$ROCKSOLID_USER

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends sudo wget curl unzip git build-essential nano less && \
    # We just install tzdata below but leave default time zone as UTC. This helps packages like Pandas to function correctly.
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata krb5-user libkrb5-dev libsasl2-dev libsasl2-modules && \
    chmod g+w /etc/passwd && \
    echo "ALL    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
    touch /etc/krb5.conf.lock && chown ${NB_USER}:${MAMBA_USER} /etc/krb5.conf* && \
    apt clean

USER $MAMBA_USER

WORKDIR "/home/${ROCKSOLID_USER}"

# Base python version will be 3.11
RUN micromamba install -y -n base -c conda-forge \
               python=3.11 \
               conda \
               jupyterlab=4.1.6 \
               ipywidgets \
               jupyterlab-lsp \
               python-lsp-server \
               matplotlib \
               pandas=2.1.4 \
               numpy=1.26.4 \
               && \
    micromamba clean --all --yes

ENV SHELL=/bin/bash
ENV EDITOR="nano"
```

### 2.2. Build image

```bash
docker build -t rocksolid-micromamba:1.5.8 .
```

### 2.3. Startup container

```bash
# Shell works
docker run -ti --rm -p 28888:8888 rocksolid-micromamba:1.5.8 /bin/bash
# Jupyterlab works, access jupyterlab from web browser http://localhost:28888
docker run -d \
           --name rocksolid-micromamba-1.5.8 \
           -p 28888:8888 \
           rocksolid-micromamba:1.5.8 jupyter-lab --no-browser --ip=0.0.0.0
```