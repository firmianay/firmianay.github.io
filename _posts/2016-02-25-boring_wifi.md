---
layout: post
title:  "记一次极度无聊下的WiFi破解和内网嗅探"
image: ''
date:   2016-02-25 00:06:31
tags:
- wifi
- wireless
description: ''
categories:
- wireless
---


> 本文写于 2016.02.25

大过年的天天走亲戚，到家第一件事就是连WiFi，唠嗑的唠嗑，玩手机的玩手机，我无聊起来也很蛋疼，突然就想看看他们都在上些什么网。。。当然了，看了自家人上网也没劲，就看看邻居家吧。。。（警察蜀黍你好，蜀黍警察再见）

思路：伪造WiFi，中间人攻击。

这种时候WiFi万能钥匙还是很好用的，一搜果然有：

![](/post_pic/2016-02-25-boring_wifi/wifi_1.jpg)

然后用个WiFi密码查看器看一眼：

![](/post_pic/2016-02-25-boring_wifi/wifi_2.jpg)

然后电脑就可以登陆了。

![](/post_pic/2016-02-25-boring_wifi/wifi_3.jpg)

打开浏览器登陆路由器看看,还连个密码都不设：

![](/post_pic/2016-02-25-boring_wifi/wifi_4.jpg)

我弱弱的伪造一个一样的WiFi，然后把原来WiFi的密码改掉，让所有人下线，由于自动连接的功能，这些设备就统统连到我伪造的WiFi上了，一般人完全看不出来。

![](/post_pic/2016-02-25-boring_wifi/wifi_5.jpg)

下面就是见证奇迹的时刻，打开wireshark抓包：

![](/post_pic/2016-02-25-boring_wifi/wifi_6.jpg)

上淘宝什么的最有爱了～当然拿到数据包，想干什么就简单了（虽然我不会。。），我就看看不说话，千万别干坏事啊。
最后说一句：WiFi有风险，蹭网需谨慎。
