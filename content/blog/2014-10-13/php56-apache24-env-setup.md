+++
title = 'PHP 5.6.0 & Apache 2.4 环境安装设置 (Ubuntu 14.04)'
subtitle = 'Setup PHP 5.6.0 & Apache 2.4 from Ubuntu 14.04'
brief = ''
date = 2014-10-13
categories = ['PHP', 'Apache', 'Ubuntu']
series = []
tags = ['PHP', 'Apache', 'Ubuntu', 'httpd']
type = 'blog'
+++

## 1. Apache编译与安装

```bash
# 安装APR依赖库
axl@MMPC:~$ sudo apt-get install libapr1-dev
axl@MMPC:~$ sudo apt-get install libaprutil1-dev

# 配置Apache源文件
axl@MMPC:~$ cd /source/software/php/httpd-2.4.10
axl@MMPC:/source/software/php/httpd-2.4.10$ ./configure --enable-so --prefix=/opt/appservers/httpd-2.4.10

# 编译Apache
axl@MMPC:/source/software/php/httpd-2.4.10$ make -s

# 安装Apache
axl@MMPC:/source/software/php/httpd-2.4.10$ sudo make install -s

# 根据--prefix配置 Apache已经被安装至/opt/appservers/httpd-2.4.10
```

- - -

## 2. PHP编译与安装

```bash
# 安装依赖库
axl@MMPC:~$ sudo apt-get install libxml2-dev
axl@MMPC:~$ sudo apt-get install libgd-dev
axl@MMPC:~$ sudo apt-get install libmcrypt-dev

# 安装freetype 2库
# download link:
# http://sourceforge.net/projects/freetype/files/freetype2/2.5.3/"
# default lib path = /usr/local/lib
axl@MMPC:~$ cd /source/software/php/
axl@MMPC:/source/software/php$ tar -xf freetype-2.5.3.tar.gz
axl@MMPC:/source/software/php$ cd freetype-2.5.3
axl@MMPC:/source/software/php/freetype-2.5.3$ ./configure
axl@MMPC:/source/software/php/freetype-2.5.3$ make -s
axl@MMPC:/source/software/php/freetype-2.5.3$ sudo make install -s

# 配置PHP源文件
./configure --prefix=/opt/appservers/php-5.6.0 --with-apxs2=/opt/appservers/httpd-2.4.10/bin/apxs --with-mysql --with-config-file-path=/opt/appservers/php-5.6.0/php.ini --with-gd --enable-gd-native-ttf --with-png --with-ttf --with-freetype-dir=/usr/local/lib --enable-mbstring --with-mcrypt --with-pdo-mysql

# 编译PHP
make -s

# 安装PHP
make install -s

# 根据--prefix配置，PHP已经被安装至/opt/appservers/php-5.6.0
# 根据--with-apxs2配置，PHP已经关联/opt/appservers/httpd-2.4.10/bin/apxs
# 根据--with-config-file-path配置，PHP配置文件指定为/opt/appservers/php-5.6.0/php.ini
```

- - -

## 3. 基本配置

```bash
# 复制默认的PHP配置文件
axl@MMPC:~$ cp /source/software/php/php-5.6.0/php.ini-development /opt/appservers/php-5.6.0/php.ini

# 编辑Apache配置文件，增加虚拟主机配置
axl@MMPC:~$ vi /opt/appservers/httpd-2.4.10/conf/httpd.conf

# 增加以下内容
-begin----------------------------------------
# Axl configure php file filter
<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>

<VirtualHost *:80>
    DocumentRoot /opt/appservers/vhosts/smd
    ServerName smdhost
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot /opt/appservers/vhosts/wphost
    ServerName wphost
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot /opt/appservers/vhosts/pshost
    ServerName pshost
</VirtualHost>

<DirectoryMatch "^/opt/appservers/vhosts/.+">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
    DirectoryIndex index.html
    DirectoryIndex index.php
</DirectoryMatch>
-end------------------------------------------
```

- - -

## 4. 启动/停止Apache

```bash
# 启动
axl@MMPC:~$ sudo /opt/appservers/httpd-2.4.10/bin/apachectl start

# 停止
axl@MMPC:~$ sudo /opt/appservers/httpd-2.4.10/bin/apachectl stop
```