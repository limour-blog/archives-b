---
title: Warp 转 SOCKS5
tags: []
id: '2121'
categories:
  - - uncategorized
date: 2023-01-30 15:46:49
---

## 申请 Warp 账户

*   下载 [wgcf](https://github.com/ViRb3/wgcf/releases)

*   wget https://github.com/ViRb3/wgcf/releases/download/v2.2.15/wgcf\_2.2.15\_linux\_amd64 -O wgcf && chmod +x wgcf

*   ./wgcf register

*   ./wgcf generate 拷贝内容如下：

```ini
[Interface]
PrivateKey = 我的私钥
Address = 172.16.0.2/32
Address = 2606:4700:110:8949:fed8:2642:a640:c8e1/128
DNS = 1.1.1.1
MTU = 1280
[Peer]
PublicKey = Warp公钥
AllowedIPs = 0.0.0.0/0
AllowedIPs = ::/0
Endpoint = engage.cloudflareclient.com:2408
```

## Warp 转 SOCKS5

```yml
version: '3.3'
services:
    socks5:
        container_name: socks5
        restart: always
        volumes:
            - './config.json:/etc/xray/config.json'
        image: teddysun/xray:1.7.2
    
networks:
  default:
    external: true
    name: warp
```

```json
{
  "dns": {
    "hosts": {
      "domain:googleapis.cn": "googleapis.com"
    },
    "servers": [
      "1.1.1.1",
  "8.8.8.8",
  "localhost"
    ]
  },
  "inbounds": [
    {
      "port": 20808,
      "protocol": "socks",
      "settings": {
        "auth": "password",
"accounts": [
{
  "user": "limour",
  "pass": "my-password"
}
],
        "udp": true,
        "userLevel": 8
      },
      "sniffing": {
        "destOverride": [
          "http",
          "tls"
        ],
        "enabled": true
      },
      "tag": "socks"
    }
  ],
  "log": {
    "loglevel": "warning"
  },
  "outbounds": [
{
  "protocol": "wireguard",
  "settings": {
"secretKey": "我的私钥",
"address": ["172.16.0.2/32", "2606:4700:110:8949:fed8:2642:a640:c8e1/128"],
"peers": [
  {
"publicKey": "Warp公钥",
"endpoint": "engage.cloudflareclient.com:2408"
  }
]
  },
  "tag": "wireguard-1"
},
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "routing": {
    "domainMatcher": "mph",
    "domainStrategy": "IPIfNonMatch",
"rules": [
{
"type": "field",
"domain": [
"geosite:cn"
],
"outboundTag": "wireguard-1"
},
{
"type": "field",
"ip": [
"geoip:cn"
],
"outboundTag": "wireguard-1"
},
{
"ip": [
  "geoip:private"
],
"outboundTag": "direct",
"type": "field"
},
{
"domain": [
  "domain:googleapis.cn"
],
"outboundTag": "direct",
"type": "field"
        }
]
  }
}
```

*   sudo docker network create warp

*   mkdir -p ~/app/socks5 && cd ~/app/socks5 && nano docker-compose.yml

*   nano ./config.json

*   sudo docker-compose up -d && sudo docker-compose logs

*   \# sudo docker-compose down

## SOCKS5 转 hysteria

```yml
version: '3.9'
services:
  hysteria:
    image: tobyxdd/hysteria
    container_name: hysteria
    restart: always
    ports:
      - '80:80'
      - '3234:3234/udp'
      - '443:443'
    volumes:
      - ./hysteria.json:/etc/hysteria.json
    command: ["server", "--config", "/etc/hysteria.json"]

```

```json
{
  "listen": ":3234",
  "protocol": "udp",
  "acme": {
    "domains": [
      "www.j11.fun"
    ],
    "email": "www@j11.fun"
  },
  "obfs": "8ZuA2Zpqhuk8yakXvMjDqEXBwY",
  "up_mbps": 100,
  "down_mbps": 100,
  "resolver": "udp://1.1.1.1:53",
  "resolve_preference": "46", 
  "socks5_outbound": {
    "server": "socks5:20808", 
    "user": "limour",
    "password": "my-password"
  }
}
```

*   mkdir -p ~/app/hysteria && cd ~/app/hysteria

*   wget https://raw.githubusercontent.com/apernet/hysteria/master/docker-compose.yaml

*   nano docker-compose.yml

*   nano ./hysteria.json

*   sudo docker-compose up -d && sudo docker-compose logs

*   sudo docker exec -it hysteria apk add curl

*   sudo docker exec -it hysteria curl --socks5 socks5:20808 --proxy-user limour:my-password http://myip.ipip.net

*   sudo docker exec -it hysteria curl --socks5 socks5:20808 --proxy-user limour:my-password https://api.ipify.org/?format=json

*   sudo docker exec -it hysteria ping socks5

*   将json文件里的socks5:20808中的socks5改为socks5对应的ip(如172.18.0.2)

*   sudo docker-compose restart

## hysteria 端口跳跃

*   iptables -t nat -L

*   模仿 docker 的操作，将 10000:60000 都映射到 hysteria 容器的3234端口

*   iptables -t nat -A DOCKER -p udp --dport 10000:60000 -j DNAT --to-destination 172.18.0.2:3234

## hysteria 客户端

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
  "server": "example.com:10000-60000",
  "obfs": "8ZuA2Zpqhuk8yakXvMjDqEXBwY",
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

*   mkdir -p ~/app/hysteria && cd ~/app/hysteria && nano docker-compose.yml

*   nano ./hysteria.json

*   sudo docker-compose up -d && sudo docker-compose logs

*   \# sudo docker-compose down

*   curl --socks5 127.0.0.1:1580 http://myip.ipip.net

*   \# sudo docker-compose restart && sudo docker-compose logs

*   curl --socks5 127.0.0.1:1580 https://api.ipify.org/?format=json