---
title: 树莓派：Docker搭建calibre-web
tags: []
id: '1888'
categories:
  - - uncategorized
date: 2022-07-11 19:33:08
---

## 来源

[https://github.com/janeczku/calibre-web](https://github.com/janeczku/calibre-web)

[https://www.bilibili.com/video/BV1CN4y137F3](https://www.bilibili.com/video/BV1CN4y137F3)

[https://hub.docker.com/r/johngong/calibre-web](https://hub.docker.com/r/johngong/calibre-web)

## 部署

```yml
version: '3.3'
services:
    calibre-web:
        container_name: calibre-web
        ports:
            - '8083:8083' # calibre-web web访问端口,默认用户名: admin 默认密码: admin123
            - '8080:8080' #calibre-server web访问端口
        volumes:
            - '/配置文件位置:/config'
            - '/书库:/library'
            - '/自动添加文件夹:/autoaddbooks'
        environment:
            - UID=1000
            - GID=1000
            - CALIBRE_SERVER_USER=用户名
            - CALIBRE_SERVER_PASSWORD=用户密码
            - CALIBRE_ASCII_FILENAME=false # (truefalse)设定false时calibre支持中文目录
        restart: unless-stopped
        image: 'johngong/calibre-web:latest'
```

*   sudo lsof -i:8080
*   sudo lsof -i:8083
*   mkdir calibre
*   cd calibre
*   nano docker-compose.yml
*   sudo docker-compose up -d

## 反代

```ini
[calibre_web]
type = http
local_ip = 127.0.0.1
local_port = 8083
use_compression = true
subdomain = calibreweb

[calibre_server]
type = http
local_ip = 127.0.0.1
local_port = 8080
use_compression = true
subdomain = calibreserver
```

## 演示

[https://j11.fun/calibreweb](https://j11.fun/calibreweb)

[https://j11.fun/calibreserver](https://j11.fun/calibreserver)

## 另一个镜像（不行，安装不上）

```yml
version: "2.1"
services:
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: calibre-web
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - DOCKER_MODS=linuxserver/calibre-web:calibre #optional
      - OAUTHLIB_RELAX_TOKEN_SCOPE=1 #optional
    volumes:
      - /path/to/data:/config
      - /path/to/calibre/library:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```

*   sudo docker-compose down 结束上一个镜像
*   开启全局代理 https://hub.docker.com/r/mzz2017/v2raya
*   mv docker-compose.yml docker-compose\_b.yml
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   拉去时可能不稳定，请静静等其自动重试，不要重新运行
*   sudo docker-compose logs
*   关闭全局代理

```sh
# run v2raya
docker run -d \
  --restart=always \
  --privileged \
  --network=host \
  --name v2raya \
  -e V2RAYA_ADDRESS=0.0.0.0:2017 \
      -v /lib/modules:/lib/modules \
  -v /etc/resolv.conf:/etc/resolv.conf \
  -v /etc/v2raya:/etc/v2raya \
  mzz2017/v2raya
```