---
title: Docker：搭建音乐播放软件Navidrome，配合Rclone访问OneDrive的歌
tags: []
id: '1902'
categories:
  - - uncategorized
date: 2022-07-12 21:01:39
---

之前利用[Rclone挂载了OneDrive到服务器上](https://occdn.limour.top/2083.html)，现在可以利用Navidrome来将其中的音乐进行管理。顺便通过RSS查看博文，发现摘要里链接太多，观感很差，因此所有链接都会附加到文字上，不直接显示，访问请点击带超链接的文字。

## 将Rclone转为自建API

*   卸载之前的挂载 fusermount -qzu /root/rclone
*   访问 [此链接注册新应用](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)，其中回调选web 重定向URL写 http://localhost
*   授予 Microsoft Graph 以下三项权限
*   Files.ReadWrite 读写用户的全部文件
*   Files.ReadWrite.All 读写用户的共享文件
*   offline\_access 可以离线访问文件
*   证书和密码里添加新客户端密码，有效期24个月，记录值，而非机密ID
*   删除 ~/.config/rclone/ 下的配置文件
*   走正常rclone流程获得配置
*   systemctl restart rclone 重启挂载

## 外部集成

*   [Last.fm](https://www.last.fm/api/account/create)
*   得到 ND\_LASTFM\_APIKEY、ND\_LASTFM\_SECRET

*   [Spotify](https://developer.spotify.com/dashboard/applications)
*   获得 ND\_SPOTIFY\_ID、ND\_SPOTIFY\_SECRET

参考1：[咕咕鸽](https://blog.laoda.de/archives/navidrome)

## 部署

```yml
version: "3"
services:
  navidrome:
    image: deluan/navidrome:latest
    ports:
      - "4533:4533"
    environment:
      # Optional: put your config options customization here. Examples:
      ND_SCANSCHEDULE: 10m
      ND_LASTFM_ENABLED:
      ND_LASTFM_APIKEY: ***
      ND_LASTFM_SECRET: ***
      ND_LASTFM_LANGUAGE: zh
      ND_SPOTIFY_ID: ***
      ND_SPOTIFY_SECRET: ***
      ND_LOGLEVEL: info  
      ND_SESSIONTIMEOUT: 24h
      ND_BASEURL: ""
      ND_ENABLETRANSCODINGCONFIG: true
      ND_TRANSCODINGCACHESIZE: 20MB
    volumes:
      - "./data:/data"
      - "../rclone/hk/Music/:/music:ro"
```

*   mkdir navidrome && cd navidrome
*   ls ../rclone/hk/Music/
*   nano docker-compose.yml
*   docker-compose up -d
*   反向代理 4533 端口
*   第一次登录时会自动进入用户创建