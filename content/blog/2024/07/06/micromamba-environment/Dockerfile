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
    apt-get install -y --no-install-recommends sudo wget curl unzip git build-essential nano less ssh && \
    # We just install tzdata below but leave default time zone as UTC. This helps packages like Pandas to function correctly.
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata krb5-user libkrb5-dev libsasl2-dev libsasl2-modules && \
    chmod g+w /etc/passwd && \
    echo "ALL    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
    touch /etc/krb5.conf.lock && chown ${ROCKSOLID_USER}:${MAMBA_USER} /etc/krb5.conf* && \
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