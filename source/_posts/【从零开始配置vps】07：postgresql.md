---
title: 【从零开始配置VPS】07：PostgreSQL
tags: []
id: '2109'
categories:
  - - uncategorized
date: 2023-01-12 15:28:58
---

```yml
version: '3.3'
services:
    db:
        container_name: postgres-db
        environment:
            - TZ=Asia/Shanghai
            - POSTGRES_USER=root
            - POSTGRES_PASSWORD=ROOT_ACCESS_PASSWORD
        ports:
            - '5432:5432'
        volumes:
            - './pgdata:/var/lib/postgresql/data'
        image: postgres
        restart: unless-stopped
```

*   mkdir -p ~/db/PostgreSQL && cd ~/db/PostgreSQL && nano docker-compose.yml

*   sudo docker-compose up -d

*   sudo docker exec -it postgres-db psql

*   使用命令 `\q` 退出psql

### 创建新数据库

*   create user ttrss with password 'ttrss\_passwd'; #创建用户ttrss

*   CREATE DATABASE ttrss OWNER ttrss; #创建用户数据库

*   GRANT ALL PRIVILEGES ON DATABASE ttrss TO ttrss; #权限都赋予ttrss