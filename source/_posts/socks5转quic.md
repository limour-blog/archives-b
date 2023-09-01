---
title: SOCKS5转QUIC
tags: []
id: '2153'
categories:
  - - uncategorized
date: 2023-03-03 11:40:49
---

## 服务端

```yml
version: '3.9'
services:
  hysteria:
    image: tobyxdd/hysteria
    container_name: hysteria
    restart: always
    ports:
      - '13234:3234/udp'
    volumes:
      - ./hysteria.json:/etc/hysteria.json
      - ./my.key:/home/ubuntu/my.key
      - ./my.pem:/home/ubuntu/my.crt
    command: ["server", "--config", "/etc/hysteria.json"]
 
networks:
  default:
    external: true
    name: sswitch
```

```json
{
  "listen": ":3234",
  "protocol": "udp",
  "cert": "/home/ubuntu/my.crt",
  "key": "/home/ubuntu/my.key",
  "obfs": "26位随机英文数字字符",
  "up_mbps": 100,
  "down_mbps": 100,
  "resolver": "udp://1.1.1.1:53",
  "resolve_preference": "46", 
  "socks5_outbound": {
    "server": "socks5:5353"
  }
}
```

*   申请证书并上传

*   mkdir -p ~/app/hysteria && cd ~/app/hysteria && nano docker-compose.yml && nano ./hysteria.json

*   sudo docker-compose up -d && sudo docker-compose logs

*   sudo docker exec -it hysteria ping socks5

*   将json文件里的socks5:5353中的socks5改为socks5对应的ip(如172.18.0.2)

*   sudo docker-compose restart

*   iptables -t nat -L

*   模仿 docker 的操作，将 32768:61000 都映射到 hysteria 容器的3234端口

*   iptables -t nat -A DOCKER -p udp --dport 32768:61000 -j DNAT --to-destination 172.19.0.4:3234

## 客户端

```yml
version: '3.9'
services:
  hysteria:
    image: tobyxdd/hysteria
    container_name: hysteria
    restart: always
    ports:
      - '1580:1580'
      - '1580:1580/udp'
      - '8580:8580'
    volumes:
      - ./hysteria.json:/etc/hysteria.json
    command: ["--config", "/etc/hysteria.json"]
```

```json
{
  "server": "证书对应的域名:32768-61000",
  "obfs": "前面的26位随机字符",
  "idle_timeout": 30,
  "hop_interval": 61,
  "up_mbps": 10,
  "down_mbps": 50,
  "socks5": {
    "listen": "0.0.0.0:1580"
  },
  "http": {
    "listen": "0.0.0.0:8580"
  }
}
```

*   mkdir -p ~/app/hysteria && cd ~/app/hysteria && nano docker-compose.yml && nano ./hysteria.json

*   sudo docker-compose up -d && sudo docker-compose logs

## 更新镜像

*   sudo docker-compose pull && sudo docker-compose up -d --remove-orphans

*   sudo docker image prune