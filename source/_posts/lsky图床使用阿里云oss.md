---
title: Lsky图床使用阿里云OSS
tags: []
id: '1912'
categories:
  - - uncategorized
date: 2022-07-16 09:47:59
---

[搭建好图床Lsky Pro](https://occdn.limour.top/2104.html)后，尝试了各种minIO的设置，死活配置不上，只好切换存储到阿里云OSS。访问域名可以用[内网机器反代](https://occdn.limour.top/2069.html)，然后套上CDF，这样可以不会消耗OSS外网流出量。顺便在用户管理，往右划拉，看到编辑按钮，这里可以更改存储容量。

![](https://img.limour.top/archives_2023/2022/07/16/62d2856a3042e.webp)

配置示例