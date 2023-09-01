---
title: Docker：部署青龙面板
tags: []
id: '1916'
categories:
  - - uncategorized
date: 2022-07-17 05:13:46
---

青龙面板是一款JavaScript/Python/Typescript/Shell脚本管理平台，可以帮助我们更方便地实现脚本的定时执行。

## 部署

```yml
version: '2'
services:
  web:
    image: whyour/qinglong:latest
    volumes:
      - ./data:/ql/data
    ports:
      - "5700:5700"
    restart: unless-stopped
```

*   mkdir qinglong && cd qinglong
*   nano docker-compose.yml
*   docker-compose up -d
*   反向代理 5700 端口
*   [获取企业微信应用](https://limour.top/1508.html)