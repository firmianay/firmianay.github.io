---
layout: post
title:  "Linux Mint 上安装 Kali 的工具"
image: ''
date:   2016-03-18 00:06:31
tags:
- linux
description: ''
categories:
- linux
---


> 本文写于 2016.03.18

由于学习渗透测试，必然要用到Kali上丰富的工具，无奈这个界面对用户太不友好，尤其是字体渲染简直不能忍，于是我想在本机Mint上安装那些工具。Ubuntu上安装应该也是可以的。

Mint基于Ubuntu，Ubuntu基于Debian，正好Kali也是基于Debian，大概可以通过添加软件源的方式交叉安装软件吧，试试看。

一、导入Kali源，选个国内比较快的。我们apt-get update后，出现问题说明：由于没有公钥，无法验证下列签名： NO_PUBKEY xxxxxxxxxxxxx 。嗯，看来要下载公钥了。

二、下载公钥。从PGP的公钥服务器上下载公钥服务器有很多个，这里是在 subkeys.pgp.net 里面。另外可能用到的还有 wwwkeys.pgp.net，如果subkeys找不到，可以换到wwwkeys看看。
```
下载： gpg --keyserver subkeys.pgp.net --recv xxxxxxxxxxxxxxxx
导入： gpg --export --armor xxxxxxxxxxxxxxxx | apt-key add -
```
这里需要在root下执行。

三、现在想安装什么都可以了。友情提醒，不要upgrade，切记，切记，切记。
