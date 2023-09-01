---
title: Nginx Proxy Manager进阶：使用Docker network
tags: []
id: '2116'
categories:
  - - uncategorized
date: 2023-01-30 04:03:47
---

## Nginx Proxy Manager

```yml
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    
networks:
  default:
    external: true
    name: ngpm
```

*   sudo docker network create ngpm

*   mkdir -p ~/base/NGPM && cd ~/base/NGPM && nano docker-compose.yml

*   sudo docker-compose up -d

*   登录到 http://ip:81

*   Email: `admin@example.com`

*   Password: `changeme`

*   反代 Nginx Proxy Manager

## 反代WordPress

```yml
version: '3.1'
services:
  wordpress:
    image: wordpress
    restart: always
    volumes:
      - ./www:/var/www/html

```

*   mkdir -p ~/app/WordPress && cd ~/app/WordPress && nano docker-compose.yml

*   sudo docker-compose up -d

*   反代写法如下 serviceName:port

![](https://img-cdn.limour.top/i/2023/01/30/63d7414ca0859.png)