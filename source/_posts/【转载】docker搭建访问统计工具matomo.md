---
title: 【转载】Docker搭建访问统计工具Matomo
tags: []
id: '1880'
categories:
  - - uncategorized
date: 2022-07-11 01:26:27
---

## 来源

咕咕鸽：https://blog.laoda.de/archives/docker-compose-install-matomo

## 部署

```yml
version: "3"

services:
  matomo_db:
    image: mariadb
    command: --max-allowed-packet=64MB
    restart: always
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=Pas3W0rd
    env_file:
      - ./db.env

  matomo_app:
    image: matomo
    restart: always
    volumes:
#     - ./config:/var/www/html/config
#     - ./logs:/var/www/html/logs
      - ./html:/var/www/html
    environment:
      - MATOMO_DATABASE_HOST=matomo_db
    env_file:
      - ./db.env
    ports:
      - 18080:80  # 8080可以更改为别的未使用的端口  lsof -i:8080 可以查看8080端口是否被使用
      - 18443:443 # 8443可以更改为别的未使用的端口  这边后续填到NPM的“Custom location”里
```

*   mkdir matomo && cd matomo
*   nano docker-compose.yml
*   nano db.env
*   docker-compose up -d
*   反代18080端口
*   将统计代码添加到主题设置里

```env
MYSQL_PASSWORD=Pas3W0rd
MYSQL_DATABASE=matomo
MYSQL_USER=matomo
MATOMO_DATABASE_ADAPTER=mysql
MATOMO_DATABASE_TABLES_PREFIX=matomo_
MATOMO_DATABASE_USERNAME=matomo
MATOMO_DATABASE_PASSWORD=Pas3W0rd
MATOMO_DATABASE_DBNAME=matomo
```