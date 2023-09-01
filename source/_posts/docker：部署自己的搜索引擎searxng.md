---
title: Docker：部署自己的搜索引擎SearXNG
tags: []
id: '1943'
categories:
  - - uncategorized
date: 2022-07-30 19:30:58
---

[咕咕鸽](https://blog.laoda.de/s/about)最近又更新了一期搭建[SearXNG](https://blog.laoda.de/archives/docker-compose-install-searxng)的教程，咱也跟着咕咕鸽的教程折腾一下

```yml
version: '3.7'
services:
  redis:
    container_name: redis_searxng
    image: "redis:alpine"
    command: redis-server --save "" --appendonly "no"
    networks:
      - searxng
    tmpfs:
      - /var/lib/redis
    cap_drop:
      - ALL
    cap_add:
      - SETGID
      - SETUID
      - DAC_OVERRIDE
 
  searxng:
    container_name: searxng
    image: searxng/searxng:latest
    networks:
      - searxng
    ports:
      - "8180:8080"   # 这个冒号左边的端口可以更改，右边的不要改
    volumes:
      - ./searxng:/etc/searxng:rw
    environment:
      - SEARXNG_HOSTNAME=s.limour.top
      - SEARXNG_BASE_URL=https://s.limour.top/
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - DAC_OVERRIDE
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"
networks:
  searxng:
    ipam:
      driver: default
```

*   mkdir -p ~/searxng && cd ~/searxng
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   sudo docker-compose logs
*   NPM面板反代8180端口，并设置域名解析。
*   sudo docker-compose down
*   sed -i "sultrasecretkey$(openssl rand -hex 32)g" searxng/settings.yml # 生成一个密钥
*   sudo docker-compose up -d