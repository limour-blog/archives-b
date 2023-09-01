---
title: 'Docker: 部署Matrix-Dendrite'
tags: []
id: '2087'
categories:
  - - uncategorized
date: 2022-12-06 15:55:22
---

[Matrix](https://github.com/matrix-org/dendrite/blob/main/build/docker/docker-compose.monolith.yml) 是一个开源、可交互、去中心化的实时通信服务框架。

## To generate keys

```sh
#/bin/bash
docker run --rm --entrypoint="" \
  -v $(pwd):/mnt \
  matrixdotorg/dendrite-monolith:latest \
  /usr/bin/generate-keys \
  -private-key /mnt/matrix_key.pem \
  -tls-cert /mnt/server.crt \
  -tls-key /mnt/server.key
```

*   mkdir -p ~/Dendrite && cd ~/Dendrite
*   nano generate-keys.sh
*   chmod +x generate-keys.sh
*   sudo ./generate-keys.sh

## 生成配置文件

*   下载 [dendrite-sample.monolith.yaml](https://github.com/matrix-org/dendrite/blob/main/dendrite-sample.monolith.yaml)
*   wget https://github.com/matrix-org/dendrite/raw/main/dendrite-sample.monolith.yaml -O dendrite.yaml
*   修改dendrite.yaml的以下配置项
*   **注意 well\_known\_server\_name 需要加端口**

```yml
global:
  server_name: m.limour.top
  private_key: /mnt/matrix_key.pem
  database:
    connection_string: postgresql://dendrite:itsasecret@postgres:5432/dendrite?sslmode=disable
  jetstream:
    storage_path: /mnt/JetStream
  well_known_server_name: "m.limour.top:443"
client_api:
  registration_shared_secret: "itsasecret"
```

## 部署

```yml
version: "3.4"
services:
  postgres:
    hostname: postgres
    image: postgres:14
    restart: always
    volumes:
      - ./create_db.sh:/docker-entrypoint-initdb.d/20-create_db.sh
      # To persist your PostgreSQL databases outside of the Docker image, 
      # to prevent data loss, modify the following ./path_to path:
      - ./postgresql:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: itsasecret
      POSTGRES_USER: dendrite
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dendrite"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - internal

  monolith:
    hostname: monolith
    image: matrixdotorg/dendrite-monolith:latest
    command: [
      "--tls-cert=/mnt/server.crt",
      "--tls-key=/mnt/server.key"
    ]
    ports:
      - 12008:8008
      - 12448:8448
    volumes:
      - ./:/etc/dendrite
      - ./media:/var/dendrite/media
      - ./:/mnt
    depends_on:
      - postgres
    networks:
      - internal
    restart: unless-stopped

networks:
  internal:
    attachable: true
```

*   nano docker-compose.yml
*   下载 [create\_db.sh](https://github.com/Limour-dev/dendrite/blob/main/build/docker/postgres/create_db.sh)
*   wget https://github.com/matrix-org/dendrite/raw/main/build/docker/postgres/create\_db.sh
*   chmod +x create\_db.sh
*   sudo docker-compose up -d

## 反向代理

*   域名解析 m.limour.top
*   WAF 添加 m.limour.top 白名单
*   NGPM反向代理 m.limour.top到12008

## 测试

*   访问 https://m.limour.top/.well-known/matrix/server
*   访问 [federationtester](https://federationtester.matrix.org/#m.limour.top)

## 注册账号

*   sudo docker exec -it dendrite-monolith-1 /usr/bin/create-account -admin --config /etc/dendrite/dendrite.yaml --username limour
*   使用 [element-web](https://app.element.io) 登录自己的服务器

## 搭建自己的element-web

```yml
version: '3.3'
services:
    element-web:
        ports:
            - '12280:80'
        volumes:
            - './config.json:/app/config.json'
            - './config.json:/app/config.e.limour.top.json'
        image: vectorim/element-web
```

```json
{
   "default_server_config": {
      "m.homeserver": {
         "base_url": "https://m.limour.top"
      },
      "m.identity_server": {
         "base_url": "https://vector.im"
      }
   }
}
```

*   mkdir -p ~/element && cd ~/element
*   nano docker-compose.yml
*   nano config.json
*   sudo docker-compose up -d
*   NPS内网穿透
*   NGPM反向代理