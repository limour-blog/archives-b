---
title: 搭建DNS服务器
tags:
  - DNS
id: '541'
categories:
  - - Raspberry Pi
date: 2021-02-22 19:28:39
---

首先安装dnsmasq

```
sudo apt-get install dnsmasq
```

修改dnsmasq.conf配置文件

```
sudo nano -K /etc/dnsmasq.conf

```

设置IGNORE\_RESOLVCONF

```
sudo nano -K /etc/default/dnsmasq
IGNORE_RESOLVCONF=yes
```

修改resolv.conf，添加上游DNS服务器

```
sudo nano -K /etc/resolv.dnsmasq.conf

```

开启dnsmasq服务

```
sudo systemctl restart dnsmasq
sudo systemctl enable dnsmasq
```

测试dnsmasq服务

```
sudo apt-get install  dnsutils

dig www.sina.cn A @127.0.0.1  grep Query

```

可以看到第二次请求的时间极大地缩短了