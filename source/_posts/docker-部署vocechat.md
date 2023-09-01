---
title: 'Docker: 部署VoceChat'
tags: []
id: '2081'
categories:
  - - uncategorized
date: 2022-12-04 18:49:20
---

`[VoceChat](https://doc.voce.chat/zh-cn/)` 是一款支持独立部署的个人云社交媒体聊天服务。15MB 的大小可部署在任何的服务器上，部署简单，很少需要维护。前端可以内嵌到自己的网站下，数据完全由用户自己掌握，传输过程加密。VoceChat 从 `Slack`, `Discord`, `RocketChat`, `Solid`, `Matrix` 等产品和规范中博采众长，适用于团队内部交流，个人聊天服务，网站客服，网站内嵌社区的场景。

## 部署

```yml
version: '3.3'
services:
    vocechat-server:
        restart: always
        ports:
            - '3009:3000'
        volumes:
            - './data:/home/vocechat-server/data'
        container_name: vocechat-server
        image: 'privoce/vocechat-server:latest'
        command: --network.frontend_url "https://c.limour.top"
```

*   mkdir -p ~/VoceChat && cd ~/VoceChat
*   nano docker-compose.yml
*   sudo docker-compose up -d

## 反向代理

*   NPS内网穿透
*   CF域名解析
*   NGPM反向代理