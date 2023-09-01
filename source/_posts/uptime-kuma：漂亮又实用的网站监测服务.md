---
title: Uptime Kuma：漂亮又实用的网站监测服务
tags: []
id: '2135'
categories:
  - - uncategorized
date: 2023-02-08 06:30:04
---

最近看到很多Alist都是在replit上白嫖的基础服务，而replit需要定时访问才能保活，因此搭建一个漂亮又实用的网站监测服务，同时还能白嫖各种需要定时访问保活的服务，岂不美哉。

```yml
version: '3.3'

services:
  uptime-kuma:
    image: louislam/uptime-kuma
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma:/app/data
    ports:
      - 3001:3001
    restart: always
```

*   mkdir -p ~/app/Uptime && cd ~/app/Uptime && nano docker-compose.yml
*   sudo docker-compose up -d
*   反代3001端口