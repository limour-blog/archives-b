---
title: Docker：部署AList V3
tags: []
id: '2114'
categories:
  - - uncategorized
date: 2023-01-29 13:08:39
---

```yml
version: '3.3'
services:
    alist:
        restart: always
        volumes:
            - './alist:/opt/alist/data'
        ports:
            - '5244:5244'
        environment:
            - PUID=0
            - PGID=0
            - UMASK=022
        container_name: alist
        image: 'xhofe/alist:latest'
```

*   mkdir -p ~/app/alist && cd ~/app/alist && nano docker-compose.yml

*   sudo docker-compose up -d

*   反代5244端口

*   获取管理员账号 `sudo docker exec -it alist ./alist admin`

*   [添加储存块](https://alist.nn.ci/zh/guide/drivers/onedrive.html)