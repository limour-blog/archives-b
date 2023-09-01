---
title: Docker：搭建图床Lsky Pro
tags: []
id: '1911'
categories:
  - - uncategorized
date: 2022-07-16 06:32:18
---

```yml
version: '3.3'
services:
    lsky-pro:
        container_name: lsky-pro
        restart: always
        volumes:
            - './lsky-pro-data:/var/www/html'
        ports:
            - '7791:80'
        image: 'dko0/lsky-pro'
```

*   mkdir lsky && cd lsky
*   nano docker-compose.yml
*   docker-compose up -d
*   反向代理 7791 端口