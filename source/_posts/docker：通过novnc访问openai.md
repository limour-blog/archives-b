---
title: Docker：通过noVNC访问OPENAI
tags: []
id: '2142'
categories:
  - - uncategorized
date: 2023-02-12 17:03:31
---

```yml
version: '3.3'
services:
    chrome-novnc:
        restart: unless-stopped
        shm_size: "512m"
        ports:
            - '6901:6080'
        environment:
            - VNCPASS=password
            - RESOLUT=1280x720
        image: x11vnc/docker-desktop:zh_CN
        volumes:
            - ./log:/home/ubuntu/.log
            - ./project:/home/ubuntu/project
        command: startvnc.sh -forever >> /home/ubuntu/.log/vnc.log
```

*   mkdir -p ~/app/novnc && cd ~/app/novnc && nano docker-compose.yml

*   mkdir log && mkdir project && chmod -R 777 log && chmod -R 777 project

*   screen docker-compose up -d

*   反代6901端口

*   访问 https://yourdomain/vnc.html?resize=downscale&autoconnect=1&password=password

![](https://img-cdn.limour.top/i/2023/02/13/63e91bbccaf5f.png)