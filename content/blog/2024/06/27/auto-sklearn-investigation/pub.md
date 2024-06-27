+++
title = 'Auto-Sklearn 调研笔记'
subtitle = 'Auto-Sklearn Investigation'
brief = ''
date = 2024-06-27T00:00:02
categories = ['Python', 'Auto-Sklearn', 'Machine Learning', 'Math']
series = ['Python Language', 'Machine Learning']
tags = ['Python', 'Auto-Sklearn', 'Machine Learning', 'Math']
type = 'blog'
+++

---

## 1. Install via Docker

---

```bash
# Get image
docker pull mfeurer/auto-sklearn:master
# Startup container
docker run -it mfeurer/auto-sklearn:master
# Startup container by interface jupyter notebook
docker run -d \
           -p 18888:8888 \
           --name auto-sklearn \
           mfeurer/auto-sklearn:master \
           /bin/bash -c "mkdir -p /opt/nb && jupyter notebook --notebook-dir=/opt/nb --ip='0.0.0.0' --port=8888 --no-browser --allow-root"
```

---

