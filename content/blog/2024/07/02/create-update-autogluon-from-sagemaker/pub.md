+++
title = '在Sagemaker distribution容器中安装或更新Autogluon环境'
subtitle = 'Create/Update Autogluon from Sagemaker distribution'
brief = ''
date = 2024-07-02T00:00:00
categories = ['Python', 'Jupyter', 'Jupyterlab', 'Sagemaker', 'Autogluon']
series = ['Python Language']
tags = ['Python', 'Jupyter', 'Jupyterlab', 'Sagemaker']
type = 'blog'
+++

---

## 1. Get image

Get images from [**AWS ECR Gallery repository**](https://gallery.ecr.aws/sagemaker/sagemaker-distribution).

```bash
docker pull public.ecr.aws/sagemaker/sagemaker-distribution:1.9.0-cpu
```

---

## 2. Startup Sagemaker container

```bash
docker run -d \
           --name sagemaker-distribution-1.9.0 \
           -p 38888:8888 \
           public.ecr.aws/sagemaker/sagemaker-distribution:1.9.0-cpu jupyter-lab --no-browser --ip=0.0.0.0
```

---

## 3. Install/Update Autogluon from Jyputerlab

Access Jupyterlab via Web browser [**http://localhost:38888**](http://localhost:38888)

Open new terminal launcher from Jupyterlab, and following below commands.

```bash
# Create new venv & active the venv.
python -m venv .u-venv-name
. .u-venv-name/bin/activate

# Install ipykernel.
python -m pip install ipykernel

# Create new ipykernel.
python -m ipykernel install --name u-ipykernel-name --display-name "You Ipykernel display name" --user

```

### 3.1. Install Autogluon for CPU environment

[**Autogluon official installation manual**](https://auto.gluon.ai/stable/install.html)

```bash
pip install -U pip
pip install -U setuptools wheel

# CPU version of pytorch has smaller footprint - see installation instructions in
# pytorch documentation - https://pytorch.org/get-started/locally/
pip install torch==2.3.1 torchvision==0.18.1 --index-url https://download.pytorch.org/whl/cpu

pip install autogluon
```

### 3.2. Install Autogluon for GPU environment

```bash
pip install -U pip
pip install -U setuptools wheel
pip install autogluon
```

## 4. Enjoy

Wait few seconds, u will found new ipykernel from Jupyterlab launcher.