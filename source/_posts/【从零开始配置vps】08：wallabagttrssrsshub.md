---
title: 【从零开始配置VPS】08：Wallabag+TTRSS+RSShub
tags: []
id: '2112'
categories:
  - - uncategorized
date: 2023-01-13 14:50:01
---

## TTRSS

```yml
version: "3"
services:
  service.rss:
    image: wangqiru/ttrss:latest
    container_name: ttrss
    ports:
      - 181:80
    environment:
      - SELF_URL_PATH=https://ttrss.limour.top/ # please change to your own domain
      - DB_HOST=172.17.0.1
      - DB_PORT=5432
      - DB_NAME=ttrss
      - DB_USER=ttrss
      - DB_PASS=ttrss_passwd
    volumes:
      - ./feed-icons:/var/www/feed-icons/
    networks:
      - public_access
      - service_only
    stdin_open: true
    tty: true
    restart: always

  service.mercury: # set Mercury Parser API endpoint to `service.mercury:3000` on TTRSS plugin setting page
    image: wangqiru/mercury-parser-api:latest
    container_name: mercury
    networks:
      - public_access
      - service_only
    restart: always

  service.opencc: # set OpenCC API endpoint to `service.opencc:3000` on TTRSS plugin setting page
    image: wangqiru/opencc-api-server:latest
    container_name: opencc
    environment:
      - NODE_ENV=production
    networks:
      - service_only
    restart: always

```

*   mkdir -p ~/app/TTRSS && cd ~/app/TTRSS && nano docker-compose.yml

*   \# 注意，TTRSS不支持MariaDB

*   sudo docker-compose up -d && sudo chmod -R 777 feed-icons

*   反代181端口

*   默认账户：admin

*   密码：password

*   偏好设置-插件里启用 mercury和opencc

*   偏好设置-订阅源-插件里填 service.mercury:3000 和 service.opencc:3000

*   [其他设置](https://wzfou.com/ttrss-docker/) 

## Wallabag

```yml
version: '3'
services:
  wallabag:
    image: wallabag/wallabag
    environment:
      - SYMFONY__ENV__FROM_EMAIL=limour@limour.top   # 修改成你自己的邮箱
      - SYMFONY__ENV__DOMAIN_NAME=https://wallabag.limour.top  # 修改成稍后要反向代理的域名
      - SYMFONY__ENV__SERVER_NAME="Limour's Wallabag"
    ports:
      - 12080:80   # 12080可以修改成其他的自己想用的端口
    volumes:
      - ./images:/var/www/wallabag/web/assets/images  # 将图片映射挂载到本地，这样docker停止了，数据不会丢失
      - ./data:/var/www/wallabag/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget" ,"--no-verbose", "--tries=1", "--spider", "http://localhost"]
      interval: 1m
      timeout: 3s
```

*   mkdir -p ~/app/Wallabag && cd ~/app/Wallabag && nano docker-compose.yml

*   sudo docker-compose up -d

*   默认登录是 `wallabag:wallabag`

## RSShub

```yml
version: '3'

```

*   mkdir -p ~/app/RSShub && cd ~/app/RSShub && nano docker-compose.yml

*   sudo docker-compose up -d

*   认证方式1：`https://rsshub.limour.top/foreverblog/feeds?key=ILoveRSSHub`

*   认证方式2：`https://usernam3:passw0rd@rsshub.limour.top/foreverblog/feeds`

## 清理Docker容器日志

```sh
#!/bin/sh 

echo "======== start clean docker containers logs ========"  

logs=$(find /var/lib/docker/containers/ -name *-json.log)  

for log in $logs  
        do  
                echo "clean logs : $log"  
                cat /dev/null > $log  
        done  

```

*   nano ~/clean\_docker\_log.sh && chmod +x ~/clean\_docker\_log.sh

*   sudo ~/clean\_docker\_log.sh