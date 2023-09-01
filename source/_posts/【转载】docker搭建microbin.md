---
title: 【转载】Docker搭建MicroBin
tags: []
id: '2111'
categories:
  - - uncategorized
date: 2023-01-13 13:31:19
---

```yml
version: '3.5'

services:
  microbin:
    image: danielszabo99/microbin:latest
    container_name: microbin
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - MICROBIN_HIGHLIGHTSYNTAX=true
      - MICROBIN_HASH_IDS=true
      - MICROBIN_EDITABLE=false
      - MICROBIN_PRIVATE=false
      - MICROBIN_HIDE_FOOTER=true
      - MICROBIN_HELP=true
      - MICROBIN_HIDE_HEADER=true
      - MICROBIN_HIDE_LOGO=true
      - MICROBIN_NO_ETERNAL_PASTA=false
      - MICROBIN_NO_FILE_UPLOAD=true
      - MICROBIN_NO_LISTING=true
      - MICROBIN_THREADS=1
      - MICROBIN_TITLE=free-bin
      - MICROBIN_PUBLIC_PATH=https://paste.limour.top/    # 记得改成自己的网址
      - MICROBIN_QR=true
    ports:
      - 18083:8080    # 冒号左边可以改成自己需要的端口号
    volumes:
      - ./microbin-data:/app/pasta_data     # 冒号左边可以改自己想要的挂载路径
```

*   mkdir -p ~/app/MicroBin && cd ~/app/MicroBin && nano docker-compose.yml
*   sudo docker-compose up -d

来源：[【好玩儿的Docker项目】10分钟搭建一个巨好用的粘贴板工具——MicroBin](https://blog.laoda.de/archives/docker-compose-install-microbin)