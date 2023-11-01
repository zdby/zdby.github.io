---
weight: 2
title: "Android镜像分析"
date: 2023-11-01T16:30:05+08:00
lastmod: 2023-11-01T16:30:05+08:00
draft: false
author: "zdby"
authorLink: "https://dillonzq.com"
description: "了解Android构建生成的镜像文件的组成."
images: []
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["content", "Markdown"]
categories: ["documentation"]

lightgallery: true

toc:
  auto: false
math:
  enable: true
---

了解如何解压Android构建生成的镜像文件，分析镜像文件包含的内容.

<!--more-->

## 1 Android构建的镜像类型

本文章是基于Android 13构建生成的镜像来分析。
虽然Android 13构建生成的镜像都是img作为文件扩展名，但是文件的实际类型却大不相同，在Linux系统下，我们用file命令来查看镜像文件的类型。
以vendor_boot.img文件为例，我们对镜像文件抽丝剥茧，一步一步把文件解压开，看看里面具体有哪些二进制文件

## 2 查看vendor_boot.img文件类型

```shell
file vendor_boot.img
```
输出结果：
> vendor_boot.img: data

## 3 解压vendor_boot.img到out目录
```shell
unpack_bootimg --boot_img vendor_boot.img --out out
```
输出结果：
> bootconfig  dtb  vendor_ramdisk  vendor_ramdisk00  vendor-ramdisk-by-name

即：在out目录下解压生成了`bootconfig`、`dtb`、`vendor_ramdisk`等5个文件

## 4 查看vendor_ramdisk文件类型

```shell
file vendor_ramdisk
```
输出结果：
> vendor_ramdisk: LZ4 compressed data (v0.1-v0.9)

即：`vendor_ramdisk`是`LZ4`类型的压缩文件

## 5 用lz4解压vendor_ramdisk
```shell
cp vendor_ramdisk vendor_ramdisk.lz4
lz4 -d vendor_ramdisk.lz4
```
输出结果：
> Decoding file vendor_ramdisk
> vendor_ramdisk.lz4   : decoded 41152000 bytes

用`lz4`命令解压文件时要求文件扩展名是lz4，所以先拷贝一份扩展名是lz4的vendor_ramdisk，解压后生成了同名文件

## 6 查看新生成的vendor_ramdisk文件类型
```shell
file vendor_ramdisk
```
输出结果：
> vendor_ramdisk: ASCII cpio archive (SVR4 with no CRC)

即：`vendor_ramdisk`是cpio归档包文件

## 7 用cpio解压vendor_ramdisk

```shell
cpio -idv < vendor_ramdisk
```
输出结果：
> first_stage_ramdisk  lib

其中`first_stage_ramdisk`目录包含了`fstab.qcom`文件
lib目录包含了众多的ko文件

> lib/modules/adsp_sleepmon.ko
> 
> lib/modules/altmode-glink.ko
> 
> lib/modules/arm_smmu.ko
> 
> ......
> 
> lib/modules/usb_f_qdss.ko
> 
> lib/modules/usbmon.ko
> 
> lib/modules/videocc-kalama.ko
