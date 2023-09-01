---
title: Docker：部署chrome-novnc
tags: []
id: '2078'
categories:
  - - uncategorized
date: 2022-11-12 06:53:56
---

## 获取镜像

*   wget https://raw.githubusercontent.com/NotGlop/docker-drag/master/docker\_pull.py

*   python3 docker\_pull.py nkpro/chrome-novnc@sha256:379ef4901f65495fb200b60fe2f87ba1346ce7df91cbe807365dba57c5dcd8d5

*   sudo apt install rclone

*   类似 [使用Rclone搭配OneDrive迁移大量数据](https://occdn.limour.top/2404.html) 的方式，将nkpro\_chrome-novnc.tar转移到服务器上

*   mkdir -p ~/.config/rclone && nano ~/.config/rclone/rclone.conf

*   rclone copy --ignore-existing --progress --ignore-errors --transfers=1 ./nkpro\_chrome-novnc.tar onedrive:tmp

*   rclone copy --ignore-existing --progress --ignore-errors --transfers=1 onedrive:tmp/nkpro\_chrome-novnc.tar .

*   docker load < nkpro\_chrome-novnc.tar

*   docker images

*   docker tag ba39b3ae6c10 nkpro/chrome-novnc:latest

## 启动镜像

```yml
version: '3.3'
services:
    chrome-novnc:
        restart: unless-stopped
        ports:
            - '15980:5980'
        volumes:
            - '/home/gene/upload:/home/user/upload'
        environment:
            - RESOLUTION=1280x720x24
        image: nkpro/chrome-novnc
        command: startVNC -forever
```

*   mkdir -p ~/zl\_liu/chrome-novnc && cd ~/zl\_liu/chrome-novnc

*   nano docker-compose.yml

*   docker-compose up -d

*   docker-compose logs

## 反向代理

*   npc内网穿透

*   ngpm反向代理，开启ws支持

*   访问  https://limour.top/vnc.html 

*   默认密码 passwd

*   修改密码 x11vnc -storepasswd passwd123 /home/user/.vnc/passwd