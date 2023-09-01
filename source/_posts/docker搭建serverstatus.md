---
title: Docker搭建ServerStatus
tags: []
id: '1997'
categories:
  - - uncategorized
date: 2022-08-27 13:39:23
---

## 服务端

```yml
version: '3.3'
services:
    serverstatus:
        restart: always
        container_name: serverstatus
        volumes:
            - './serverstatus-config.json:/ServerStatus/server/config.json'
            - './serverstatus-monthtraffic:/usr/share/nginx/html/json'
        ports:
            - '8001:80'
            - '35601:35601'
        image: 'cppla/serverstatus:latest'
```

*   mkdir -p ~/sss && cd ~/sss
*   wget --no-check-certificate -qO ./serverstatus-config.json https://raw.githubusercontent.com/cppla/ServerStatus/master/server/config.json && mkdir ./serverstatus-monthtraffic
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   NPM面板反代8001端口，并设置[域名解析](https://tz.limour.eu.org/)。

## 客户端

```json
{
"servers": [
{
"username": "ld_rasp4",
"name": "ld_rasp4",
"type": "arm",
"host": "LD",
"location": "CN",
"password": "DEFAULT_PASSWORD",
"monthstart": 1
},
{
"username": "oc_blog",
"name": "oc_blog",
"type": "amd",
"host": "🇸🇬",
"location": "US",
"password": "DEFAULT_PASSWORD",
"monthstart": 1
}
],
"watchdog": [
]
}
```

*   编辑 serverstatus-config.json
*   重启服务端 sudo docker-compose restart
*   wget --no-check-certificate -qO client-linux.py 'https://gogs.frp.limour.top/limour/ServerStatus/raw/master/clients/client-linux.py'
*   chmod +x client-linux.py
*   测试一下：/home/pi/client-linux.py SERVER=45.79.67.132 USER=ld\_rasp4 PASSWORD=DEFAULT\_PASSWORD INTERVAL=10
*   创建system服务
*   sudo nano /etc/systemd/system/ssc.service
*   sudo systemctl enable ssc
*   sudo systemctl start ssc
*   sudo systemctl status ssc

```conf
[Unit]
Description=ServerStatus-Client
After=network.target
[Service]
ExecStart=/home/pi/client-linux.py SERVER=45.79.67.132 USER=ld_rasp4 PASSWORD=DEFAULT_PASSWORD INTERVAL=10
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
```