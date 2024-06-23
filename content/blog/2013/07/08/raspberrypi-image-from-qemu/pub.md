+++
title = '使用QEMU仿真程序运行Raspberry Pi镜像'
subtitle = 'Raspberry Pi image running from QEMU'
brief = ''
date = 2013-07-08
categories = ['Linux', 'Arm', 'QEMU', 'Raspberry Pi']
series = []
tags = ['Arm', 'Linux', 'Kernel', 'SOC', 'QEMU', 'Raspberry Pi']
type = 'blog'
+++

## 1. 工具 (Tools)

>[1] Host linux version: Ubuntu 12.04 LTS (Linux 3.2.0-29-generic-pae)

>[2] Raspberry Pi image 3.6.8

>[3] QEMU: QEMU emulator version 1.5.0

- - -

## 2. 操作步骤 (Steps)
### 2.1. 启动 (Startup)

```bash
# Raspberry Pi image file is 2013-05-29-wheezy-armel.zip
unzip 2013-05-29-wheezy-armel.zip
# 2013-05-29-wheezy-armel.img

qemu-system-arm \
-kernel kernel-qemu \
-cpu arm1176 \
-m 256 \
-M versatilepb \
-no-reboot \
-serial stdio \
-append "root=/dev/sda2 panic=1" \
-hda 2013-05-29-wheezy-armel.img \
-net nic,macaddr=00:1a:80:d2:0b:3f,addr=192.168.9.11 \
-net tap,ifname=tap0,script=no
# Above the tap network is optional.
```

### 2.2. 改变镜像文件大小 (Resize image file)

```bash
# From host machine
qemu-img resize 2013-05-29-wheezy-armel.img +1024M # (1024 * n)
# Startup QEMU, follow 2.1 section
```

```bash
# From of QEMU shell
# Review partition information
df -h
# /dev/sda is image file, or is naming like /dev/mmcblk0, which will be represent different storage type.
fdisk /dev/sda
# Type command p for display partition information as below.
p
# Disk /dev/sda: 8382 MB, 8382316544 bytes 
# 64 heads, 32 sectors/track, 7994 cylinders, total 16371712 sectors 
# Units = sectors of 1 * 512 = 512 bytes 
# Sector size (logical/physical): 512 bytes / 512 bytes 
# I/O size (minimum/optimal): 512 bytes / 512 bytes 
# Disk identifier: 0x00099f20 
#  
# Device Boot         Start       End      Blocks   Id  System 
# /dev/sda1            8192    122879       57344    c  W95 FAT32 (LBA) 
# /dev/sda2          122880    1xxxxx     2124416   83  Linux 
#  
# Command (m for help):

# The 122880 is start sector of partition, record that number for below.
# Type command d for remove partition 2 follow the tips of console.
d
2
# Type p to review partition information again.

# Disk /dev/sda: 8382 MB, 8382316544 bytes 
# 64 heads, 32 sectors/track, 7994 cylinders, total 16371712 sectors 
# Units = sectors of 1 * 512 = 512 bytes 
# Sector size (logical/physical): 512 bytes / 512 bytes 
# I/O size (minimum/optimal): 512 bytes / 512 bytes 
# Disk identifier: 0x00099f20 
#  
#    Device Boot      Start         End      Blocks   Id  System 
# /dev/sda1            8192      122879       57344    c  W95 FAT32 (LBA) 
#  
# Command (m for help):

# Type n for new partition, Partition type will be P(Primary), and Partition number will be 2.

# Type p to review partition information again.

# Disk /dev/sda: 8382 MB, 8382316544 bytes 
# 64 heads, 32 sectors/track, 7994 cylinders, total 16371712 sectors 
# Units = sectors of 1 * 512 = 512 bytes 
# Sector size (logical/physical): 512 bytes / 512 bytes 
# I/O size (minimum/optimal): 512 bytes / 512 bytes 
# Disk identifier: 0x00099f20 
#  
#    Device Boot      Start         End      Blocks   Id  System 
# /dev/sda1            8192      122879       57344    c  W95 FAT32 (LBA) 
# /dev/sda2          122880    16371711     8124416   83  Linux 
#  
# Command (m for help):

# If all done, type w to make the partition table rewrite.

# Don't do anything, have to reboot now.
reboot

# When the reboot done, need to resize file system as below.
# The device reference maybe naming as /dev/mmcblk0p2, it determinated by different device type.
resize2fs /dev/sda2
```
