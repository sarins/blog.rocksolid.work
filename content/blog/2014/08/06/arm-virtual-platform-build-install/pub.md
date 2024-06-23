+++
title = '虚拟ARM开发平台编译与安装'
subtitle = 'ARM virtual platform building & install'
brief = ''
date = 2014-08-06
categories = ['Linux', 'Arm', 'C', 'Make']
series = []
tags = ['Arm', 'Linux', 'Kernel', 'SOC', 'QEMU', 'Cross Compile']
type = 'blog'
+++

## 1. 工具 (Tools)

>[1] Host linux version: Ubuntu 12.04 LTS (Linux 3.2.0-29-generic-pae)

>[2] Target linux kernel: linux-2.6.32.61.tar.xz

>[3] QEMU: QEMU emulator version 1.0.50 (Debian 1.0.50-2012.03-0ubuntu2), Copyright (c) 2003-2008 Fabrice Bellard

>[4] Linux cross compiler: arm-none-linux-gnueabi-gcc (Sourcery CodeBench Lite 2013.05-24) 4.7.3 (Site: http://www.mentor.com/embedded-software/codesourcery)

>[5] Mac OS 10.9 (Site: http://www.carlson-minot.com/available-arm-eabi-g-lite-builds-for-mac-os-x)

>[6] U-boot: u-boot-2013.04.tar.bz2

>[7] Busybox: busybox-1.21.0.tar.bz2

>[8] Genext2fs: genext2fs-1.4.1.tar.gz


- - -

## 2. 工具的安装 (Tools install)

### 2.1. 默认的环境设置 (Default enviroument setup)

```bash
export ARCH=arm
export CORSS_COMPILE=” arm-none-linux-gnueabi-”
```

You can make the shell script like below

```bash
#!/bin/bash

export PATH=/usr/CodeSourcery/Sourcery_CodeBench_Lite_for_ARM_GNU_Linux/bin/:/usr/u-boot-2013.04/tools/:$PATH
export ARCH=arm
export CROSS_COMPILE=" arm-none-linux-gnueabi-"
```

- - -

### 2.2. 安装QEMU (QEMU install)

> 下载 (Download) [http://wiki.qemu.org/Download](http://wiki.qemu.org/Download)

```bash
wget xxxxx_location/qemu-x.x.x.tar.bz2
tar -xvf qemu-x.x.x.tar.bz2
cd  qemu-x.x.x
chmod 777 /opt (if u want install the qemu to /opt/qemu, u will need this operation)
./configure --prefix=/opt/qemu --target-list=arm-softmmu,arm-linux-user –enable-debug
make -s
make install -s
```

- - -

### 2.3. 安装交叉编译工具 (arm-none-linux-gnueabi [CROSS_COMPILER])

> 下载 (Download) [http://www.codesourcery.com/sgpp/lite/arm/portal/subscription?@template=lite](http://www.codesourcery.com/sgpp/lite/arm/portal/subscription?@template=lite)

```bash
wget arm-2013.05-24-arm-none-linux-gnueabi.bin
chmod +x arm-2013.05-24-arm-none-linux-gnueabi.bin
./arm-2013.05-24-arm-none-linux-gnueabi.bin
graphic setting & install
```

- - -

### 2.4. 安装U-Boot (U-Boot install)

> 下载 (Download) &nbsp;[ftp://ftp.denx.de/pub/u-boot/](ftp://ftp.denx.de/pub/u-boot/)

```bash
wget u-boot-2013.04.tar.bz2
tar -xvf u-boot-2013.04.tar.bz2
cd u-boot-2013.04
make versatilepb_config
make
```
- - -

## 3. 操作步骤 (Steps)
### 3.1. 编译内核 (Compile kernel)

```bash
export ARCH=arm
export CORSS_COMPILE=” arm-none-linux-gnueabi-”
tar -xvf linux-2.6.32.61.tar.xz
make versatile_defconfig
make menuconfig
```

> choose “Kernel Features → Use the ARM EABI to compile the kernel”

> choose “Kernel Features → Allow old ABI binaries to run with this kernel (EXPERIMENTAL)

> choose “File Systems → Second extended fs support”

> choose “General setup → Initial RAM filesystem and RAM disk (initramfs/initrd) supporte ”

> choose “Device Drivers → Block Devices → Default RAM disk size (kbytes)  = 4096”

```bash
make -s (if use u-boot, make uImage -s)
```

- - -

### 3.2. 创建初始化内存磁盘 (Create RAM disk)

#### 设置环境变量 (Setup enviroument var)

```bash
export ARCH=arm
export CORSS_COMPILE=” arm-none-linux-gnueabi-”
```

#### 创建根文件系统 (Create root file system)

```bash
#GOTO workspace_folder (any where)
mkdir rootfs
cd rootfs
mkdir bin dev etc lib proc sbin sys usr mnt tmp var
mkdir usr/bin usr/lib usr/sbin lib/modules
mknod -m 666 dev/console c 5 1
mknod -m 666 dev/null c 1 3
ln -s dev/null dev/tty2
```

#### 编译内核以及模块并安装到根文件系统中 (Compile kernel/moudles and install it to root file system)

```bash
#GOTO kernel folder
make modules
make modules_install INSTALL_MOD_PATH=workspace_folder/rootfs
```

#### 编译并安装Busybox (Compile & install busybox)

```bash
#GOTO BusyBox source folder (example: /usr/busybox-1.21.0)
make defconfig
make menuconfig
```

> choose “Busybox Settings → Build Options → Build BusyBox as a static binary (no shared libs) ”

> choose “Busybox Settings → Build Options → Cross Compiler prefix = arm-none-linux-gnueabi-” (optional)

> choose “Busybox Settings → General Configuration → Don't use /usr”

> choose “Busybox Settings → Installation Options →  BusyBox installation prefix =workspace_folder/rootfs”


```bash
make
make install
```

#### 从Busybox复制最小设备文件 (Copy minimal etc files from busybox)

```bash
cp –a busibox_source/examples/bootfloppy/etc/* workspace_folder/rootfs/etc
```

#### 使用genext2fs将根文件系统打包为ext2格式 (use genext2fs package rootfs to ext2 file)

```bash
#GOTO workspace_folder
genext2fs -b 4096 -d rootfs ramdisk
```

- - -

### 3.3. 启动仿真程序 (Startup emulator)

```bash
#GOTO linux_kernel_folder
qemu-system-arm -M versatilepb -kernel arch/arm/boot/zImage -initrd /home/axl/ramdisk -append "root=/dev/ram init=/linuxrc"
```
