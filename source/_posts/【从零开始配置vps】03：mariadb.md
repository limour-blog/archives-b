---
title: 【从零开始配置VPS】03：MariaDB
tags: []
id: '2098'
categories:
  - - uncategorized
date: 2023-01-03 13:46:32
---

```yml
version: "2.1"
services:
  mariadb:
    image: linuxserver/mariadb:latest
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=ROOT_ACCESS_PASSWORD
      - TZ=Asia/Shanghai
    volumes:
      - ./config:/config
    ports:
      - 3306:3306
    restart: unless-stopped
```

*   mkdir -p ~/db/MariaDB && cd ~/db/MariaDB

*   nano docker-compose.yml

*   sudo docker-compose up -d