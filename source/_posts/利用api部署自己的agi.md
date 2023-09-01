---
title: 利用API部署自己的AGI
tags: []
id: '2156'
categories:
  - - uncategorized
date: 2023-03-04 03:02:53
---

简单记录一下[chatgpt-demo](https://github.com/ddiu8081/chatgpt-demo)这个项目的部署过程。

```Dockerfile
# ---- Dependencies ----
FROM node:19.7.0
WORKDIR /app
COPY chatgpt-demo /app
RUN npm config set registry https://registry.npmmirror.com \
 && npm install \
 && rm ./src/pages/api/generate.ts
ENV PORT=3000
CMD ["npm", "run", "dev", "--", "--port", "3000", "--host", "0.0.0.0"]
```

```yml
version: '3.3'
services:
    chatgpt:
        container_name: chatgpt
        restart: always
        image: limour/chatgpt
        ports:
            - '6903:3000'
        volumes:
            - ./.env:/app/.env
            - ./generate.ts:/app/src/pages/api/generate.ts
```

*   mkdir -p ~/app/chatGPT && cd ~/app/chatGPT && nano Dockerfile && nano docker-compose.yml
*   git clone --depth=1 https://ghproxy.com/https://github.com/ddiu8081/chatgpt-demo.git
*   cp ./chatgpt-demo/.env.example .env && nano .env # 填上自己的key
*   cp ./chatgpt-demo/src/pages/api/generate.ts generate.ts && nano generate.ts # 改成自己反代的api地址
*   自己反代的api的方法：[利用Golang反代某AI的API](https://occdn.limour.top/2607.html)
*   sudo docker build -t limour/chatgpt .
*   sudo docker-compose up -d && sudo docker-compose logs
*   反代 6903 端口