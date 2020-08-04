---
layout: post
title:  "在Ubuntu上安装 JDK 8"
image: ''
date:   2016-01-02 00:06:31
tags:
- linux
description: ''
categories:
- linux
---


> 本文写于 2016.01.02

Ubuntu默认的 JDK 是 OpenJDK，兼容性不太好，下面我们来安装Oracle JDK。
首先确认下是否已经安装了JDK：
```
$ javac -version
```

卸载 OpenJDK：
```
$ sudo apt-get purge openjdk-*
```

由于卸载后可能有残留，导致运行 `java -version` 时第一行不是 java 版本号，而是 `Picked up JAVA_TOOL_OPTIONS: -javaagent:/usr/share/java/jayatanaag.jar` 这个提示，导致很多检测java版本号的脚本运行出错，因此需要手动清除残留。
```
sudo rm /usr/share/upstart/sessions/jayatana.conf
```

一、上oracle官网下载适合版本的 JDK。

二、我们把 JDK 安装到”`/usr/local/java`” (或者ubuntu默认地址 `/usr/lib/jvm`)。首先创建目录：
```
$ cd /usr/local
$ sudo mkdir java
```

解压下载的 JDK 包：
```
$ cd /usr/local/java
$ sudo tar xzvf ~/Downloads/jdk-8u{xx}-linux-x64.tar.gz
       // x: extract, z: for unzipping gz, v: verbose, f: filename
```

三、告诉ubuntu使用这个 JDK/JRE：
```
// Setup the location of java, javac and javaws
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.8.0_{xx}/jre/bin/java" 1
      // --install symlink name path priority
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.8.0_{xx}/bin/javac" 1
$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/jdk1.8.0_{xx}/jre/bin/javaws" 1

// Use this Oracle JDK/JRE as the default
$ sudo update-alternatives --set java /usr/local/java/jdk1.8.0_{xx}/jre/bin/java
      // --set name path
$ sudo update-alternatives --set javac /usr/local/java/jdk1.8.0_{xx}/bin/javac
$ sudo update-alternatives --set javaws /usr/local/java/jdk1.8.0_{xx}/jre/bin/javaws
```
上面几步是在目录 `/usr/bin` 中建立 java, javac, javaws 的系统链接，链接目录 `/etc/alternatives` 里的文件 。（为了解决多个版本的 JDK 共存的问题）
```
$ cd /usr/bin
$ ls -ld java*
lrwxrwxrwx 1 root root 22 Mar 31 20:41 java -> /etc/alternatives/java
lrwxrwxrwx 1 root root 23 Mar 31 20:42 javac -> /etc/alternatives/javac
lrwxrwxrwx 1 root root 24 Mar 31 20:42 javaws -> /etc/alternatives/javaws

$ cd /etc/alternatives
$ ls -ld java*
lrwxrwxrwx 1 root root 40 Aug 29 18:18 java -> /usr/local/java/jdk1.8.0_20/jre/bin/java
lrwxrwxrwx 1 root root 37 Aug 29 18:18 javac -> /usr/local/java/jdk1.8.0_20/bin/javac
lrwxrwxrwx 1 root root 42 Aug 29 18:19 javaws -> /usr/local/java/jdk1.8.0_20/jre/bin/javaws
```

四、确认 JDK 已经安装：
```
// Show the Java Compiler (javac) version
$ javac -version
javac 1.8.0_20

// Show the Java Runtime (java) version
$ java -version
java version "1.8.0_20"
Java(TM) SE Runtime Environment (build 1.8.0_20-b26)
Java HotSpot(TM) 64-Bit Server VM (build 25.20-b23, mixed mode)

// Show the location of javac and java
$ which javac
/usr/bin/javac
$ which java
/usr/bin/java
```

另外还有一种安装的方法，比较简单，但是容易出问题，特别是网络不好的时候（说多了都是泪）。
```
sudo apt-get purge openjdk*

sudo apt-get install software-properties-common

sudo add-apt-repository ppa:webupd8team/java

sudo apt-get update

sudo apt-get install oracle-java8-installer
```
可能出现下面类似的错误：
```
正在保存至: “jdk-7u51-linux-x64.tar.gz”

     0K                                                      100% 1.06M=0.005s

2014-03-16 16:57:20 (1.06 MB/s) - 已保存 “jdk-7u51-linux-x64.tar.gz” [5307/5307])

Download done.
Removing outdated cached downloads...
sha256sum mismatch jdk-7u51-linux-x64.tar.gz
Oracle JDK 7 is NOT installed.
dpkg：处理 oracle-java7-installer (--configure)时出错：
 子进程 已安装 post-installation 脚本 返回了错误号 1
正在设置 gsfonts-x11 (0.22) ...
在处理时有错误发生：
 oracle-java7-installer
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

问题就是文件下载失败，长度获取错误，有2个解决办法：

1、输入 cd /var/cache 进入 apt-cache 查看是否存在一个类似于 oracle-jdk8-installer 的文件夹，如果有就直接输入 sudo rm -rf oracle-jdk8-installer 整个删除，然后重新尝试上面的安装步骤。

2、进入 oracle-jdk8-installer 文件夹将 oracle-jdk8-installer.tar.gz 文件删除，去 Oracle 官网下载 oracle-jdk8-installer.tar.gz 放到这个文件夹中 sudo mv ~/downloads/oracle-jdk8-installer.tar.gz /var/cache/oracle-jdk8-installer 移动位置。最后需要输入 sudo dpkg –configure -a 修复一下 dpkg 配置信息。
