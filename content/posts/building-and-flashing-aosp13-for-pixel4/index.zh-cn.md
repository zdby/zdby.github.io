---
weight: 2
title: "aosp13编译及pixel4刷机指南"
date: 2023-11-03T10:30:05+08:00
lastmod: 2023-11-03T10:30:05+08:00
draft: false
author: "zdby"
authorLink: "https://zdby.github.io"
description: "了解aosp13编译构建的方法及用生成的镜像给pixel4刷机的方法."
images: []
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["aosp13", "pixel4"]
categories: ["documentation"]

lightgallery: true

toc:
  auto: false
math:
  enable: true
---

了解aosp13编译构建的方法、了解pixel4刷机的方法

<!--more-->

## 1 简介

本文章主要介绍下载aosp13源码，编译构建aosp13，并将生成的镜像包给pixel4刷机的方法。

主要分如下三个步骤：
1. 下载aosp13源码
2. 下载驱动包
3. 构建及刷机

## 2 下载aosp13源码
下载aosp13源码前，需要先在如下谷歌官网查询pixel4支持的aosp13版本号。

aosp13是一个大版本，包含了许多各个tag号的小版本，不同的小版本支持的设备不一样。

[build号对应的版本tag(标记)及支持的设备](https://source.android.google.cn/docs/setup/about/build-numbers?hl=zh-cn)

在这个页面搜`Pixel 4`，可以看到tag号为`android-13.0.0_r1`的版本支持pixel4，结果如下。
|build号 | 标记 | 版本 | 支持的设备 | 安全补丁级别
|:------:|:------:|:------:|:------:| ---------:|
| TP1A.220624.014	| android-13.0.0_r1	| Android13	| Pixel 4、Pixel 4a、Pixel 4a (5G)、Pixel 4 XL、Pixel 5、Pixel 5a (5G)	| 2022-08-05 |

下载aosp13源码的命令（这里用国内中科大的镜像源）
```Shell
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-13.0.0_r1
```

## 3 下载驱动包
要在实体机pixel4上刷机，还需要下载谷歌及高通的驱动包才能正常开机。

刚才查询版本`android-13.0.0_r1`对应的build号是`TP1A.220624.014`。

我们在官网[pixel4驱动包](https://developers.google.cn/android/drivers?hl=zh-cn#flametp1a.220624.014)搜索`TP1A.220624.014`对应的驱动包，

下载2个驱动包后放到aosp13源码的根目录并解压，可以得到2个sh文件：
```Shell
# 解压命令
~/code/aosp$ tar zxvf *.tgz -C ./
# 解压后的文件
~/code/aosp$ ls -alh *.sh
-rwxr-x--x 1 admin admin 416M Aug  3  2022 extract-google_devices-flame.sh
-rwxr-x--x 1 admin admin 1.2M Aug  3  2022 extract-qcom-flame.sh
```
然后执行这两个sh脚本（需要同意协议），就可以把驱动包拷贝到对应目录下

## 4 构建及刷机
在源码根目录下执行命令，构建aosp13
```Shell
source build/envsetup.sh
lunch aosp_flame-userdebug
make -j32
```
经过几个小时的编译后，在目录`out/target/product/flame`下会生成镜像包，把该目录下的img文件及`android-info.txt`拷贝到windows电脑的D盘out目录下。

```Shell
~/code/aosp/out/target/product/flame$ ls -alh *.img android-info.txt
-rw-rw-r-- 1 admin admin   26 Oct 29 21:46 android-info.txt
-rw-rw-r-- 1 admin admin  38M Oct 30 10:41 boot-debug.img
-rw-rw-r-- 1 admin admin  64M Oct 30 10:41 boot.img
-rw-r--r-- 1 admin admin 8.7M Oct 30 10:41 bootloader.img
-rw-rw-r-- 1 admin admin  38M Oct 30 10:41 boot-test-harness.img
-rw-rw-r-- 1 admin admin 1.0M Oct 29 21:46 dtb.img
-rw-rw-r-- 1 admin admin 8.0M Oct 29 22:06 dtbo.img
-rw-rw-r-- 1 admin admin 276M Oct 30 10:41 product.img
-rw-r--r-- 1 admin admin  81M Oct 30 10:41 radio.img
-rw-rw-r-- 1 admin admin  17M Oct 30 10:41 ramdisk-debug.img
-rw-rw-r-- 1 admin admin  581 Oct 29 22:06 ramdisk.img
-rw-rw-r-- 1 admin admin  16M Oct 30 10:41 ramdisk-recovery.img
-rw-rw-r-- 1 admin admin  17M Oct 30 10:41 ramdisk-test-harness.img
-rw-r--r-- 1 admin admin 4.8K Oct 30 10:41 super_empty.img
-rw-rw-r-- 1 admin admin 160M Oct 30 10:41 system_ext.img
-rw-rw-r-- 1 admin admin 840M Oct 30 10:41 system.img
-rw-rw-r-- 1 admin admin  25M Oct 30 10:41 system_other.img
-rw-r--r-- 1 admin admin 2.6M Oct 29 22:13 userdata.img
-rw-rw-r-- 1 admin admin 4.0K Oct 30 10:41 vbmeta.img
-rw-rw-r-- 1 admin admin 4.0K Oct 30 10:41 vbmeta_system.img
-rw-r--r-- 1 admin admin 837M Oct 30 10:41 vendor.img
```

在`cmd`中设置环境变量
```Shell
set ANDROID_PRODUCT_OUT=D:\out
```

然后在`cmd`中用`fastboot flashall -w`命令开始刷机
```Shell
C:\Users\admin>fastboot flashall -w
--------------------------------------------
Bootloader Version...: c2f2-0.4-8048765
Baseband Version.....: g8150-00123-220122-B-8106568
Serial Number........: 9BxxxxxxxxxxN5
--------------------------------------------
Checking 'product'                                 OKAY [  0.070s]
Setting current slot to 'a'                        OKAY [  0.079s]
Sending 'boot_a' (65536 KB)                        OKAY [  0.381s]
Writing 'boot_a'                                   OKAY [  0.451s]
Sending 'dtbo_a' (8192 KB)                         OKAY [  0.146s]
Writing 'dtbo_a'                                   OKAY [  0.126s]
Sending 'vbmeta_a' (4 KB)                          OKAY [  0.143s]
Writing 'vbmeta_a'                                 OKAY [  0.063s]
Sending 'vbmeta_system_a' (4 KB)                   OKAY [  0.147s]
Writing 'vbmeta_system_a'                          OKAY [  0.077s]
Rebooting into fastboot                            OKAY [  0.068s]
< waiting for any device >
Sending 'super' (4 KB)                             OKAY [  0.010s]
Updating super partition                           OKAY [  0.018s]
Resizing 'product_a'                               OKAY [  0.000s]
Resizing 'system_a'                                OKAY [  0.000s]
Resizing 'system_ext_a'                            OKAY [  0.000s]
Resizing 'system_b'                                OKAY [  0.016s]
Resizing 'vendor_a'                                OKAY [  0.000s]
Resizing 'vendor_b'                                OKAY [  0.003s]
Resizing 'product_a'                               OKAY [  0.000s]
Sending sparse 'product_a' 1/2 (262140 KB)         OKAY [  0.982s]
Writing 'product_a'                                OKAY [  1.375s]
Sending sparse 'product_a' 2/2 (19160 KB)          OKAY [  0.082s]
Writing 'product_a'                                OKAY [  0.174s]
Resizing 'system_a'                                OKAY [  0.000s]
Sending sparse 'system_a' 1/4 (262116 KB)          OKAY [  1.012s]
Writing 'system_a'                                 OKAY [  1.388s]
Sending sparse 'system_a' 2/4 (262116 KB)          OKAY [  1.003s]
Writing 'system_a'                                 OKAY [  1.379s]
Sending sparse 'system_a' 3/4 (262140 KB)          OKAY [  0.952s]
Writing 'system_a'                                 OKAY [  1.429s]
Sending sparse 'system_a' 4/4 (70892 KB)           OKAY [  0.263s]
Writing 'system_a'                                 OKAY [  0.453s]
Resizing 'system_ext_a'                            OKAY [  0.009s]
Sending 'system_ext_a' (163420 KB)                 OKAY [  0.669s]
Writing 'system_ext_a'                             OKAY [  0.879s]
Resizing 'system_b'                                OKAY [  0.039s]
Sending 'system_b' (24708 KB)                      OKAY [  0.118s]
Writing 'system_b'                                 OKAY [  0.166s]
Resizing 'vendor_a'                                OKAY [  0.000s]
Sending sparse 'vendor_a' 1/4 (262116 KB)          OKAY [  1.005s]
Writing 'vendor_a'                                 OKAY [  1.387s]
Sending sparse 'vendor_a' 2/4 (262096 KB)          OKAY [  0.961s]
Writing 'vendor_a'                                 OKAY [  1.386s]
Sending sparse 'vendor_a' 3/4 (262140 KB)          OKAY [  0.978s]
Writing 'vendor_a'                                 OKAY [  1.368s]
Sending sparse 'vendor_a' 4/4 (67004 KB)           OKAY [  0.273s]
Writing 'vendor_a'                                 OKAY [  0.443s]
Erasing 'userdata'                                 OKAY [  5.470s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Erasing 'metadata'                                 OKAY [  0.051s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Rebooting                                          OKAY [  0.000s]
Finished. Total time: 47.554s
```

经过`48s`的刷机，pixel4就可以正常开机，由此开启真机调试`framework`代码的旅程了~