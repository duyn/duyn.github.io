---
layout: post
title:  "Ubuntu下使用QEMU模拟Cortex-A9编译运行u-boot"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
  feature:
date:   2017-03-22 23:49:54
---

* Table of Contents
{:toc}

## 安装交叉工具链

```shell
$sudo apt-get insatll gcc-arm-linux-gnueabi
$sudo apt-get insatll g++-arm-linux-gnueabi 
```
注意：此处arm-linux-gnueabi-gdb没有被安装，可以自己编译一个:
1. 下载 gdb（我用的是gdb-7.5.1.tar.bz2）	2. 安装 libncurses5-dev（termcap库在这里）
   ```shell
   $ sudo apt-get install libncurses5-dev
   ```
2. 解压配置
   ```shell
   $ tar xzvf /mnt/hgfs/share/gdb-7.5.1.tar.bz2 -C ./
   $ cd gdb-7.5.1
   $ ./configure --target=arm-linux-gnueabi
   ```
3. 安装（默认地址 /usr/local）
   ```shell
   $ make
   $ sudo make install
   ```
4. sudo ldconfig系统配置生效
5. 可以使用了 


## 安装Qemu 
```shell
$ sudo apt-get install qemu qemu-system qemu-utils
```

## 下载u-boot最新版
## 编译uboot
1. 解压
   ```shell
   $ tar jxvf /mnt/hgfs/share/u-boot-latest.tar.bz2 -C ./
   ```
2. 解压后进入源代码目录，在Makefile里面增加两行：
   ```shell
   ARCH ?= arm
   CROSS_COMPILE ?= arm-linux-gnueabi-
   ```
   其实就是告诉它使用ARM编译器来编译。
3. 编译
   ```shell
   $ make vexpress_ca9x4_config
   ```
   注：vexpress_ca9x4 通过include/configs/下的头文件得到
   ```shell
   $ make
   ```
4. 运行
   ```shell
   qemu-system-arm -M vexpress-a9 -m 256M -nographic -kernel u-boot
   ```