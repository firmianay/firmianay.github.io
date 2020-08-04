---
layout: post
title:  "Linux下制作u盘启动盘"
image: ''
date:   2015-12-29 00:06:31
tags:
- linux
description: ''
categories:
- linux
---


> 本文写于 2015.12.29

作为不折腾会死星人，重装系统是常有的事，特别是入了Linux的坑的话，制作u盘的系统安装盘就很有必要了。

第一种方法：如果你是Ubuntu用户的话，自带就有一个工具Startup Disk Creator，做出来的优盘和原装光盘效果完全一样，读取更快还能保存修改。缺点是只能制作Ubuntu的及其衍生版，如果在Ubuntu环境下制作Ubuntu启动盘，这个方法当是首推。

第二种方法：unetbootin工具。这个工具在Linux、Windows下都可以一样地使用。自带下载光盘镜像的功能，不过为了版本较新和保证速度，还是先下载好吧。unetbootin实际上是在u盘里头装了一个Grub，与dd命令一样不能保存修改。

1、先安装工具
```
sudo apt-get install unetbootin
```

2.格式化u盘
```
sudo fdisk -l #查看U盘盘符，假设为/dev/sdb
sudo umount /dev/sdb #先卸载u盘
sudo mkfs.vfat /dev/sdb #格式化为fat32模式
```

3.使用unetbootin制作u盘镜像

第三种方法：dd命令，这是我最常使用的方法。dd可以读取磁盘设备的内容（几乎是直接读取扇区），然后将整个设备备份成一个文件。这里我们用kali linux做示范。

首先要将u盘取消挂载
```
umount /dev/sdb1
```
然后dd命令烧录
```
sudo dd bs=4M if=~/linux_images/kali-linux-1.0-amd64.iso of=/dev/sdb && sync
```
解释一下：
```
bs=BYTES：read and write up to BYTES bytes at a time
if=FILE：read from FILE instead of stdin
of=FILE：write to FILE instead of stdout
sync：Force changed blocks to disk, update the super block.
```
注：dd命令中的目标是sdb，没有标号。

当然如果你想回到Windows，可以使用winusb工具，需要通过PPA来安装，用Debian的小伙伴们就不能享用了。。。

先安装：
```
sudo add-apt-repository ppa:colingille/freshlight
sudo apt-get update
sudo apt-get install winusb
```
打开软件，第一步选择将要使用的windows ISO镜像文件，然后在target device里面选择U盘，点击INSTALL，几分钟后windows U盘启动盘就制作好了。

WINUSB也支持命令行模式，输入：
```
sudo winusb --format <iso path> <device>  //格式化U盘
sudo winusb --install <iso path> <partition> //制作启动盘
```
