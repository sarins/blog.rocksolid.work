+++
title = 'Linux ramfs & tmpfs 基础操作'
subtitle = 'Linux ramfs & tmpfs basis operations'
brief = ''
date = 2015-02-26
categories = ['Linux', 'Ramfs', 'Tmpfs', 'Tips']
series = []
tags = ['Linux', 'Ramfs', 'Tmpfs']
type = 'blog'
+++

## 1. 挂载ramfs

```bash
# 基于root权限操作，建立挂载点。
[root@MMPC ~]# mkdir -p /mnt/test_ramfs

# 挂载ramfs。
[root@MMPC ~]# sudo mount -t ramfs -o size=32m ramfs /mnt/test_ramfs
```

- - -

## 2. 挂载tmpfs

```bash
# 基于root权限操作，建立挂载点。
[root@MMPC ~]# mkdir -p /mnt/test_tmpfs

# 挂载tmpfs。
[root@MMPC ~]# sudo mount -t tmpfs -o size=32m tmpfs /mnt/test_tmpfs
```

- - -

## 3. 特性对比

| 特性     | ramfs | tmpfs |
| -------- | ----- | ----- |
| 固定大小 | No    | Yes   |
| 使用Swap | No    | Yes   |
| 持久存储 | No    | No    |