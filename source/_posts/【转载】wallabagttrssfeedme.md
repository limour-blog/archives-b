---
title: 【转载】Wallabag+TTRSS+FeedMe
tags: []
id: '1881'
categories:
  - - uncategorized
date: 2022-07-11 04:33:27
---

## 来源

咕咕鸽：https://blog.laoda.de/archives/docker-compose-install-wallabag

## TTRSS部署

*   mkdir ttrss && cd ttrss
*   wget https://raw.githubusercontent.com/HenryQW/Awesome-TTRSS/main/docker-compose.yml
*   修改 PostgreSQL 数据库的默认密码
*   修改 Tiny Tiny RSS 服务的部署网址修改为我们实际的部署网址
*   docker-compose up -d
*   反代181端口
*   默认账户：admin
*   密码：password
*   终止容器 docker-compose down && docker volume prune && rm -r data/
*   偏好设置-插件里启用 mercury和opencc
*   偏好设置-订阅源-插件里填 service.mercury:3000 和 service.opencc:3000
*   其他设置 https://wzfou.com/ttrss-docker/
*   添加知乎每日精选进行测试 https://www.zhihu.com/rss
*   同时可以在RSShub找到订阅源 https://docs.rsshub.app/usage.html#sheng-cheng-ding-yue-yuan

![](https://img-cdn.limour.top/blog/20220711110841.png)

效果很棒！

## Wallabag部署

```yml
version: '3'
services:
  wallabag:
    image: wallabag/wallabag
    environment:
      - SYMFONY__ENV__FROM_EMAIL=limour@limour.top   # 修改成你自己的邮箱
      - SYMFONY__ENV__DOMAIN_NAME=https://wallabag.j11.fun  # 修改成稍后要反向代理的域名
      - SYMFONY__ENV__SERVER_NAME="Limour's Wallabag"
    ports:
      - 12080:80   # 12080可以修改成其他的自己想用的端口
    volumes:
      - ./images:/var/www/wallabag/web/assets/images  # 将图片映射挂载到本地，这样docker停止了，数据不会丢失
      - ./data:/var/www/wallabag/data
    healthcheck:
      test: ["CMD", "wget" ,"--no-verbose", "--tries=1", "--spider", "http://localhost"]
      interval: 1m
      timeout: 3s
```

*   mkdir wallabag && cd wallabag
*   nano docker-compose.yml
*   docker-compose up -d
*   反代 12080
*   默认登录是`wallabag:wallabag`
*   安装浏览器拓展 Wallabagger
*   新建一个专有用户，创建应用API
*   在TTRSS中添加 这个的订阅源
*   使用教程：https://blog.laoda.de/archives/docker-compose-install-wallabag

![](https://img-cdn.limour.top/blog/20220711121934.png)

效果很棒！

## FeedMe使用

*   在TTRSS偏好设置里生成APP密码
*   勾选偏好设置-通用里的启用API
*   谷歌市场安装FeedMe，TTRSS登录，域名需要带https://前缀
*   登录后点击同步

![](https://img-cdn.limour.top/blog/f8968e59b82a59b69db5bc99a31f345.jpg)

效果很棒！