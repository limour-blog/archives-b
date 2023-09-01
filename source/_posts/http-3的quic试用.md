---
title: HTTP/3的QUIC试用
tags: []
id: '2060'
categories:
  - - uncategorized
date: 2022-10-20 07:05:26
---

```yml
version: '3.3'
services:
    quic:
        container_name: quic
        ports:
            - '52014:443/udp'
        volumes:
            - './:/cert/'
            - './server.json:/root/server.json'
        image: 'hmmabc/tuic-server:0.8.5'
        restart: unless-stopped
```

*   mkdir quic && cd quic
*   上传证书的pem和key
*   \# apt install dos2unix
*   \# dos2unix 1.key
*   \# dos2unix 1.pem
*   nano docker-compose.yml
*   nano server.json
*   sudo docker-compose up -d
*   sudo docker-compose logs
*   \# 再次遇到某手机无法使用的问题。。。

```json
{
    "port": 443,
    "token": ["example"],
    "certificate": "/cert/1.pem",
    "private_key": "/cert/1.key",
    "ip": "0.0.0.0",
    "congestion_controller": "bbr",
    "alpn": ["h3"]
}
```