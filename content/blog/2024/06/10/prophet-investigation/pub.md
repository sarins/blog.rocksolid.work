+++
title = 'Facebook Prophet Docker环境准备工作'
subtitle = 'Facebook Prophet Docker environment preparation'
brief = ''
date = 2024-06-10T00:00:00
categories = ['Prophet', 'Anaconda']
series = ['Prophet']
tags = ['Prophet', 'Anaconda', 'Docker', 'Python']
type = 'blog'
+++

## 1. 准备工作 (Preparation)

### 1.1. Docker镜像 (Docker image)

```dockerfile
# Dockerfile based on miniconda, jupyter & prophet will be preinstalled.
FROM continuumio/miniconda3:24.4.0-0

EXPOSE 9900
WORKDIR /opt
# RUN /bin/bash
#RUN conda install -c conda-forge jupyter -y --quiet
#RUN conda install -c conda-forge prophet -y --quiet
RUN conda install -c conda-forge jupyter -y
RUN conda install -c conda-forge prophet -y
# RUN conda install -c conda-forge plotly -y

RUN mkdir -p /opt/notebooks
CMD ["jupyter", "notebook", "--notebook-dir=/opt/notebooks", "--ip='*'", "--port=9900", "--no-browser", "--allow-root"]
```

> [Dockerfile download](../Dockerfile)

### 1.2. 编译镜像 (Compile docker image)

```bash
# The tag of image will be 'miniconda-jupyter-prophet:24.4.0-0'.
docker build -t miniconda-jupyter-prophet:24.4.0-0 .
```

### 1.3. 启动容器 (Startup container)

```bash
# Startup container (with volume mount)
docker run -d --name miniconda-jupyter-prophet -p 59900:9900 -v /opt/docker_data/prophet-notebooks:/opt/notebooks miniconda-jupyter-prophet:24.4.0-0

# Startup container (with out volume mount)
docker run -d --name miniconda-jupyter-prophet -p 59900:9900 miniconda-jupyter-prophet:24.4.0-0
```

### 1.4. 容器初始化 (Init container)
```bash
# Install deps inside of container [this will be move into Dockerfile]
conda install -c conda-forge plotly --solver=classic
# Fixed issue of conda command always tip error message like "Error while loading conda entry point: conda-libmamba-solver (libarchive.so.19: cannot open shared object file: No such file or directory)"
conda install --solver=classic conda-forge::conda-libmamba-solver conda-forge::libmamba conda-forge::libmambapy conda-forge::libarchive
```

> Make sure all of dependencies coming from same channel of pip
