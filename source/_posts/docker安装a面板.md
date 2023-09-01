---
title: Docker安装A面板
tags: []
id: '1999'
categories:
  - - uncategorized
date: 2022-09-01 14:58:58
---

之前[安装了X面板](https://occdn.limour.top/2001.html)，现在在客户端上安装A面板来使用它

```yml
version: '3.3'
services:
    v2raya:
        restart: always
        privileged: true
        network_mode: host
        container_name: v2raya
        environment:
            - 'V2RAYA_ADDRESS=0.0.0.0:2017'
        volumes:
            - '/lib/modules:/lib/modules'
            - '/etc/resolv.conf:/etc/resolv.conf'
            - './v2raya:/etc/v2raya'
        image: mzz2017/v2raya
```

*   mkdir va && cd va
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   远程桌面浏览器访问虚拟机ip:2017