---
layout: post
title:  "快速建博客指南"
image: ''
date:   2015-11-09 00:06:31
tags:
- linux
description: ''
categories:
- linux
---

> 本文写于 2015.11.09

本人也是小白一个，从零开始摸索，走了些弯路，把它记录下来，希望对大家有些帮助。

本地系统kali2.0，服务器是阿里云ubuntu14.04，web环境搭建使用LNMP一键安装包，博客系统wordpress，域名firmianay.com。

LNMP一键安装包是一个用Linux Shell编写的可以为CentOS/RadHat/Fedora、Debian/Ubuntu/RaspbianVPS(VDS)或独立主机安装LNMP(Nginx/MySQL/PHP)、LNMPA(Nginx/MySQL/PHP/Apache)、LAMP(Apache/MySQL/PHP)生产环境的Shell程序。附上下载地址 ：下载

WordPress是一种使用PHP语言开发的博客平台，用户可以在支持PHP和MySQL数据库的服务器上架设属于自己的网站。也可以把 WordPress当作一个内容管理系统（CMS）来使用。 下载

下载完成后，我们把这两个文件上传到服务器端，使用sftp工具。sftp类似于ftp，但它进行加密传输，有更高的安全性。在命令行模式下输入sftp root@firmianay.com，输入密码，提示下载一个证书，回车确定，就进入sftp>了。输入 put lnmp1.2-full.tar.gz ，回车，等待完成。WordPress也一样， put wordpress-4.3.1-zh_CN.zip 。

接下来我们使用另一个工具Putty，Putty是一款远程登录工具，用它可以非常方便的登录到Linux服务器上进行各种操作。输入IP，端口号，服务等，点击打开之后输入用户名、密码就登陆到服务器了。

输入 tar zxf lnmp1.2-full.tar.gz && cd lnmp1.2-full && ./install.sh lnmp 。如需要安装LNMPA或LAMP，将./install.sh 后面的参数替换为lnmpa或lamp即可。接下来是各种选项，如果不确定就一路回车。安装时间比较长，当然也是看服务器的配置了。

安装完成，输入lnmp vhost add 添加虚拟主机，输入 unzip wordpress-4.3.1-zhCN.zip 解压，把 wordpress 里面的文件复制到虚拟主机中，mv * /home/wwwroot/firmianay.com 。

打开浏览器输入firmianay.com ，就开始配置网站了，特别注意把表前缀 wp 改为 dm_ ，不是必须的，但是推荐改。点击安装之后就搞定了～

但是有时候后台上传附件时提示“上级目录没有写权限”，首先确保以上提示中的网站目录是存在并且访问权限是可以写入的，如果问题不是这个，就输入 chown -R www /home/wwwroot/firmianay.com 改所属者，这个方法同时也解决了直接在后台更新插件或安装插件，填写ftp用户名和密码后，提示不正确的问题。

推荐插件：

Jetpack，这个应该是必装的，可以开启Markdown。

Crayon Syntax Highlighter，很好用的代码高亮插件，关键是与Jetpack完全兼容。

BackWPup，用来给网站备份的。
