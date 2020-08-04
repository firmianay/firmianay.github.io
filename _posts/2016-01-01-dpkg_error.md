---
layout: post
title:  "ubuntu apt-get dpkg 遇到的问题及解决方法"
image: ''
date:   2016-01-01 00:06:31
tags:
- linux
description: ''
categories:
- linux
---


> 本文写于 2016.01.01

一、有时候我们在用sudo apt-get install 安装软件时，由于各种特殊的原因，直接关闭了终端，但进程没有结束，结果终端提示
```
E: 无法获得锁 /var/lib/dpkg/lock – open (11: 资源暂时不可用)
E: 无法锁定管理目录(/var/lib/dpkg/)，是否有其他进程正占用它？
```
解决办法如下：

1 终端输入 ps aux ，找到含有apt-get的进程，直接sudo kill PID。

2 强制解锁

```
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
```
然后因 dpkg 被中断，需要输入这个来恢复
```
sudo dpkg --configure -a
```

二、我在安装 oracle-java8 的时候遇到了这种情况（往往与上一个问题一起出现）
```
debconf: DbDriver "config": /var/cache/debconf/config.dat is locked by another process: 资源暂时不可用
正在设置 oracle-java8-installer (8u66+8u65arm-1~webupd8~1) ...
debconf: DbDriver "config": /var/cache/debconf/config.dat is locked by another process: 资源暂时不可用
dpkg: 处理软件包 oracle-java8-installer (--configure)时出错：
子进程 已安装 post-installation 脚本 返回了错误号 1
在处理时有错误发生：
oracle-java8-installer
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

解决方法：
```
sudo lsof /var/cache/debconf/config.dat
```
得到下面的提示：
```
lsof: WARNING: can't stat() fuse.gvfsd-fuse file system /run/user/1000/gvfs
      Output information may be incomplete.
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
frontend 16507 root    4uW  REG    8,1    42243 2888840 /var/cache/debconf/config.dat
```
杀死进程，然后就可以进行一开始的安装：
```
sudo kill 16507
sudo apt-get install oracle-java8-installer
```
