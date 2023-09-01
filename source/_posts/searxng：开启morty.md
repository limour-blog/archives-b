---
title: SearXNG：开启morty
tags: []
id: '2132'
categories:
  - - uncategorized
date: 2023-02-06 13:32:36
---

类似[WebVPN](https://webvpn.fudan.edu.cn/)，[morty项目](https://github.com/asciimoo/morty)可以实现网址替换，将访问 `baidu` 或者其它网站的请求进行代理，更具体的可以看[zwc的介绍](https://zwc365.com/2020/09/02/morty-server)。

morty是SearXNG的一个插件，之前[部署过SearXNG](https://occdn.limour.top/2177.html)，这里记录一下如何开启这个功能。

## 部署morty

```yml
version: '3.3'
services:
    morty:
        environment:
            - DEBUG=false
            - 'MORTY_ADDRESS=0.0.0.0:3000'
        image: dalf/morty
        restart: always
        ports:
            - '3000:3000'
        command: -timeout 60 -ipv6=false
```

*   mkdir -p ~/app/morty && cd ~/app/morty && nano docker-compose.yml
*   sudo docker-compose up -d
*   反代3000端口
*   为了安全，[建议开启认证](https://occdn.limour.top/2566.html)

## 开启morty

*   cd /root/app/SearXNG
*   sudo nano /root/app/SearXNG/searxng/settings.yml
*   取消 morty 相关的注释
*   sudo docker-compose restart

## 附加

上次部署时忘记配置redis了，这里顺便记录一下配置方式

*   sudo nano /root/app/SearXNG/searxng/settings.yml
*   将redis项下的`url: false`改为`url: redis://redis:6379/0`