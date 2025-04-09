+++
title = '开发工作站设置'
subtitle = 'Dev Station Configuration'
brief = ''
date = 2025-03-07T00:00:00
categories = ['PVE', 'Apache', 'Database', 'OSS', 'MLflow', 'Docker']
series = ['Dev Environment']
tags = ['PVE', 'Apache', 'Database', 'OSS', 'MLflow', 'Docker', 'Minio', 'PGVector', 'PostgreSQL', 'MySQL']
type = 'blog'
+++

---

# 1. Dev Station Spec

---

## 1.1. Hardware/Software Specification
| Component            | Model                       |      Vendor |           Specifications |
|----------------------|-----------------------------|------------:|-------------------------:|
| CPU                  | AMD Ryzen™ 7 8845HS         |         AMD |    3.8-5.1 GHz / 8 Cores |
| Memory               | Crucial 96GB DDR5-5600      |     Crucial |       48GBx2 SODIMM 1.1V |
| HD                   | SAMSUNG 990 EVO 2T          |     SAMSUNG |   NVMe / PCIe4.0*4/5.0*2 |
| AI Engine            | AMD Ryzen™ AI               |         AMD |                  38 TOPS |
| NPU                  | AMD Ryzen™ AI               |         AMD |                  16 TOPS |
|                      |                             |             |                          |
| Host Environment     | Proxmox VE 8.3              |     Proxmox |         QEMU / KVM / LXC |

---

## 1.2. Host Domain & Port

| Application          | Domain                      |     Port(s) |                       IP |
|----------------------|-----------------------------|------------:|-------------------------:|
| PVE                  | pve.rocksolid.work          |         N/A |               172.88.0.2 |
| OpenWRT/Lede         | openwrt.rocksolid.work      |         N/A |             172.88.0.254 |
| Ubuntu               | ubuntu.rocksolid.work       |         N/A |              172.88.0.11 |

---

## 1.3. Services Domain & Port

| Application          | Domain                      |     Port(s) |                 Features |
|----------------------|-----------------------------|------------:|-------------------------:|
| Apache               | N/A, ONLY Work as Web proxy |      80/443 |                    Proxy |
| MLflow               | mlflow.rocksolid.work       |       65000 | Tracing / Model Registry |
| MySQL                | mysql.rocksolid.work        |        3306 |                    RDBMS |
| Minio                | minio.rocksolid.work        |   9000/9090 |                  OSS(S3) |
| PostgreSQL           | minio.rocksolid.work        |   5432/5432 |                    RDBMS |
| PGVector             | minio.rocksolid.work        |   6432/5432 |            Vector Search |
| Keycloak             | keycloak.rocksolid.work     |  48080/8080 |             OAuth & SAML |
| Casdoor              | casdoor.rocksolid.work      |  48000/8000 |             OAuth & SAML |
| ActiveMQ             | activemq.rocksolid.work     | 61616/61616 |                       MQ |
| Redis-Stack          | redis.rocksolid.work        |   6379/6379 |                    Redis |


| Application          | Domain                      |     Port(s) |                 Features |
|----------------------|-----------------------------|------------:|-------------------------:|
| LobeChat             | lobe.rocksolid.work         |   3210/3210 |   Knowledge Base & Agent |
| ...                  | na.rocksolid.work           |     .../... |                      ... |

---

# 2. Setup Scenario

---

## 2.1 Docker IP-VLAN Level 3 Setup

---

> Dcoker IP-VLAN will be setup as 192.168.11.0/24

```bash
# Create IP-VLAN for docker environment.
docker network create \
    --driver ipvlan \
    --subnet=192.168.11.0/24 \
    --gateway=192.168.11.1 \
    -o parent=ens18 \
    -o ipvlan_mode=l3 \
    #-o ipvlan_flag=bridge \
    #-o com.docker.network.bridge.enable_icc=true \
    #-o com.docker.network.bridge.enable_ip_masquerade=true \
    rs_vlan

# Run container for testing.
docker run --net=rs_vlan -it --rm busybox /bin/sh
docker run --net=rs_vlan -it --rm --ip=172.88.1.128 busybox /bin/sh
```

---

## Optional Install Portainer Dashboard
```bash
# Create volume
docker volume create portainer_data
# Startup container
docker run -d \
           --name portainer \
           -v /var/run/docker.sock:/var/run/docker.sock \
           -v portainer_data:/data \
           --net=rs_vlan \
           --ip=192.168.11.254 \
           portainer/portainer-ce:2.27.3
```

---

## 2.2. Fundamental Service Container(s) Setup

---

### 2.2.1. MySQL
```bash
docker run -d \
           --name mysql-server \
           --env MYSQL_ROOT_PASSWORD="6yhn*IK<" \
           --env MYSQL_ROOT_HOST=% \
           --net=rs_vlan \
           --ip=192.168.11.5 \
           mysql/mysql-server:8.0.15
```

---

### 2.2.2. Redis Stack
```bash
docker run -d \
           --name redis-stack \
           --net=rs_vlan \
           --ip=192.168.11.6 \
           redis/redis-stack:6.2.6-v9
```

---

### 2.2.3. ActiveMQ
```bash
docker run -d \
           --name activemq \
           --net=rs_vlan \
           --ip=192.168.11.7 \
           apache/activemq-classic:5.18.7
```

##### ActiveMQ Ports

> * ActiveMQ WebConsole on `8161`
> * ActiveMQ JMX MBean server on `1099`
> * ActiveMQ tcp connector on `61616`
> * ActiveMQ AMQP connector on `5672`
> * ActiveMQ STOMP connector on `61613`
> * ActiveMQ MQTT connector on `1883`
> * ActiveMQ WS connector on `61614`

##### ActiveMQ Environment Variables

| Environment Variable           | Description                                                                                                                                     |
|--------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `ACTIVEMQ_CONNECTION_USER`     | Username to access transport connector on the broker (JMS, ...). If not set, no user and password are required                                  |
| `ACTIVEMQ_CONNECTION_PASSWORD` | Password to access transport connector on the broker (JMS, ...). It should be used with `ACTIVEMQ_CONNECTION_USER`.                             |
| `ACTIVEMQ_JMX_USER`            | Username to access the JMX MBean server of the broker. If set, ActiveMQ accepts remote JMX connection, else, only local connection are allowed. |
| `ACTIVEMQ_JMX_PASSWORD`        | Password to access the JMX MBean server of the broker. It should be used with `ACTIVEMQ_JMX_USER`/                                              |
| `ACTIVEMQ_WEB_USER`            | Username to access the ActiveMQ WebConsole.                                                                                                     |
| `ACTIVEMQ_WEB_PASSWORD`        | Password to access the ActiveMQ WebConsole.                                                                                                     |

---

### 2.2.4. Minio
```bash
docker run -d \
           --name minio \
           -e "MINIO_ROOT_USER=root" \
           -e "MINIO_ROOT_PASSWORD=6yhn*IK<" \
           --net=rs_vlan \
           --ip=192.168.11.8 \
           minio/minio:RELEASE.2025-04-03T14-56-28Z server /data --console-address ":80"
```

---

## 2.3. ML Service Container(s) Setup

---

### 2.3.1. Jupyterlab Based On Micromamba

---

> Dockerfile-micromamba
```dockerfile
# Base micromamba image
FROM mambaorg/micromamba:2.0.8-cuda12.2.2-ubuntu22.04

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
    apt-get install -y --no-install-recommends sudo wget curl unzip git build-essential nano less ssh openssh-server net-tools iputils-ping && \
    # We just install tzdata below but leave default time zone as UTC. This helps packages like Pandas to function correctly.
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends tzdata krb5-user libkrb5-dev libsasl2-dev libsasl2-modules && \
    chmod g+w /etc/passwd && \
    echo "ALL    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
    touch /etc/krb5.conf.lock && chown ${ROCKSOLID_USER}:${MAMBA_USER} /etc/krb5.conf* && \
    apt clean

USER $MAMBA_USER

WORKDIR "/home/${ROCKSOLID_USER}"

COPY global-gitconfig /home/${ROCKSOLID_USER}/.gitconfig

ENV SHELL=/bin/bash
ENV EDITOR="nano"
```

> Build Command
```bash
# Execute the command in the directory where the "Dockerfile-micromamba" file is located.
docker build -t rocksolid/micromamba:2.0.8-cuda12.2.2-ubuntu22.04 -f ./Dockerfile-micromamba .
```

> Startup Container
```bash
# Below container(s) also belong to the rs_vlan(Docker IP-VLAN Level 3);
# The volume "nfs-shared" will be used to share storage data files between these containers;
# Python 3.10 environment including jupyter-lab, which is service at port 80.
docker run -d \
           --name python3.10-micromamba \
           -v /opt/nfs-shared:/opt/nfs-shared \
           --net=rs_vlan \
           --ip=192.168.11.9 \
           rocksolid/micromamba:2.0.8-cuda12.2.2-ubuntu22.04 /bin/bash -c "\
           micromamba install -y -n base -c conda-forge \
                      python=3.10 \
                      jupyterlab \
                      ipywidgets \
                      jupyterlab-lsp \
                      python-lsp-server && \
           jupyter-lab --notebook-dir /opt/nfs-shared \
                       --no-browser \
                       --ip=0.0.0.0 \
                       --port=80"

# Python 3.12 environment including jupyter-lab, which is service at port 80.
docker run -d \
           --name python3.10-micromamba \
           -v /opt/nfs-shared:/opt/nfs-shared \
           --net=rs_vlan \
           --ip=192.168.11.10 \
           rocksolid/micromamba:2.0.8-cuda12.2.2-ubuntu22.04 /bin/bash -c "\
           micromamba install -y -n base -c conda-forge \
                      python=3.12 \
                      jupyterlab \
                      ipywidgets \
                      jupyterlab-lsp \
                      python-lsp-server && \
           jupyter-lab --notebook-dir /opt/nfs-shared \
                       --no-browser \
                       --ip=0.0.0.0 \
                       --port=80"
```

---

### 2.3.2. MLflow Service
```bash
docker run -d \
           --name mlflow \
           --net=rs_vlan \
           --ip=192.168.11.11 \
           ghcr.io/mlflow/mlflow:v2.21.3 mlflow server --host 0.0.0.0 --port 80
```


---

## 2.4. LLM & Knowledge Base Service Container(s) Setup (TODO)

---

## 2.5. Remote Development Container(s) Setup (TODO)

---

<!--

### 2.2.n. PGVector

```bash
docker run -d \
           --name pgvector-16 \
           -p 6432:5432 \
           --env POSTGRES_USER=postgres \
           --env POSTGRES_PASSWORD="6yhn*IK<" \
           pgvector/pgvector:pg16
```

```bash
# Casdoor
# Image: docker pull casbin/casdoor:v1.857.0
docker run -d \
           --name casdoor-v1.857.0 \
           -p 48000:8000 \
           -e httpport=8000 \
           -e RUNNING_IN_DOCKER=true \
           -e driverName=postgres \
           -e dataSourceName="user=postgres password=6yhn*IK< host=localhost port=6432 sslmode=disable dbname=casdoor" \
           -e runmode=dev \
           -e origin="http://localhost:3210" \
           -v /Users/axl/dev/tmp/init_data.json:/init_data.json \
           casbin/casdoor:v1.857.0
```

```bash
# Genrate rand secret (UdZZOhacRAknWuiEp9zJ0C9HD0zf+E9tOFX7HUOx3pM=) via openssl
openssl rand -base64 32
# LobeChat Server mode
docker run -d \
           --name lobe-chat-server \
           -p 3210:3210 \
           -e APP_URL=http://10.1.2.228:3210 \
           -e DATABASE_URL="postgres://postgres:6yhn*IK<@host.docker.internal:6432/lobe" \
           -e KEY_VAULTS_SECRET=UdZZOhacRAknWuiEp9zJ0C9HD0zf+E9tOFX7HUOx3pM= \
           -e NEXT_AUTH_SECRET=+b0OerJVFp9V4ojVWgr8tLMXln1QG8HxZFdSEgdvHHc= \
           -e NEXT_AUTH_SSO_PROVIDERS=casdoor \
           -e AUTH_CASDOOR_ID=a387a4892ee19b1a2249 \
           -e AUTH_CASDOOR_SECRET=dbf205949d704de81b0b5b3603174e23fbecc354 \
           -e AUTH_CASDOOR_ISSUER=http://10.1.2.228:48000 \
           -e NEXTAUTH_URL=http://10.1.2.228:3210/api/auth \
           -e S3_ACCESS_KEY_ID=ySpo8jOW4ZjyBFsfCEGO \
           -e S3_SECRET_ACCESS_KEY=dKOEYanJUovlkckasssE72viFQmnOcpwDDDNgMHz \
           -e S3_ENDPOINT=http://10.1.2.228:9000 \
           -e S3_BUCKET=lobe \
           -e S3_PUBLIC_DOMAIN=http://10.1.2.228:9000 \
           lobehub/lobe-chat-database:1.68.10


docker run -d \
           --name lobe-chat-server \
           -p 3210:3210 \
           -e DATABASE_URL="postgres://postgres:6yhn*IK<@host.docker.internal:6432/lobe" \
           -e KEY_VAULTS_SECRET=UdZZOhacRAknWuiEp9zJ0C9HD0zf+E9tOFX7HUOx3pM= \
           -e APP_URL=http://localhost:3210 \
           -e S3_ACCESS_KEY_ID=ySpo8jOW4ZjyBFsfCEGO \
           -e S3_SECRET_ACCESS_KEY=dKOEYanJUovlkckasssE72viFQmnOcpwDDDNgMHz \
           -e S3_ENDPOINT=http://localhost:9000 \
           -e S3_BUCKET=lobe \
           -e S3_PUBLIC_DOMAIN=http://localhost:9000 \
           lobe-chat-database-local:latest
```

-->