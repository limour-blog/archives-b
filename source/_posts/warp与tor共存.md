---
title: Warp与Tor共存
tags: []
id: '2151'
categories:
  - - uncategorized
date: 2023-03-02 16:36:28
---

先搭建好Tor，方法[点此](https://occdn.limour.top/2596.html)。

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

## Warp与Tor自动切换

```yml
version: '3.3'
services:
    socks5:
        container_name: socks5
        restart: always
        volumes:
            - './config/:/etc/sing-box/'
        image: gzxhwq/sing-box:git
    
networks:
  default:
    external: true
    name: sswitch
```

```json
{
  "log": {
    "level": "info"
  },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "::",
      "listen_port": 5353,
      "tcp_fast_open": false,
      "udp_fragment": false,
      "sniff": true,
      "sniff_override_destination": false,
      "sniff_timeout": "300ms",
      "udp_timeout": 300,
      "proxy_protocol": false,
      "proxy_protocol_accept_no_header": false
    }
  ],
  "outbounds": [
    {
      "type": "direct"
    },
    {
      "type": "wireguard",
      "tag": "wireguard",
      "server": "engage.cloudflareclient.com",
      "server_port": 2408,
      "local_address": ["10.0.0.2/32","fd00::1/128"],
      "private_key": "我的私钥",
      "peer_public_key": "Warp公钥"
    },
    {
      "type": "socks",
      "tag": "tor",
      "server": "tor",
      "server_port": 9050
    }
  ],
  "route": {
    "rules": [
      {
        "domain_suffix": [".onion"],
        "outbound": "tor"
      },
      {
        "domain_suffix": ["openai.com"],
        "outbound": "wireguard"
      },
      {
        "domain_suffix": [".cn"],
        "outbound": "wireguard"
      },
      {
        "domain_suffix": ["check.torproject.org"],
        "outbound": "tor"
      },
      {
        "domain_suffix": ["myip.ipip.net"],
        "outbound": "wireguard"
      },
      {
        "geoip": ["cn"],
        "outbound": "wireguard"
      },
      {
        "geosite": ["cn"],
        "outbound": "wireguard"
      }
    ]
  }
}
```

*   mkdir -p ~/app/socks5 && cd ~/app/socks5 && nano docker-compose.yml
*   mkdir ./config
*   nano ./config/config.json
*   sudo docker-compose up -d && sudo docker-compose logs

## 测试warp

*   docker run --rm --net=sswitch alpine/curl http://myip.ipip.net
*   docker run --rm --net=sswitch alpine/curl --socks5 socks5:5353 http://myip.ipip.net
*   docker run --rm --net=sswitch alpine/curl --socks5 socks5:5353 https://api.ipify.org/?format=json

## 测试tor

*   docker run --rm --net=sswitch alpine/curl --socks5-hostname "tor:9050" "https://check.torproject.org/api/ip"
*   docker run --rm --net=sswitch alpine/curl --socks5-hostname "socks5:5353" "https://check.torproject.org/api/ip"
*   docker run --rm --net=sswitch alpine/curl --socks5-hostname "tor:9050" "https://www.nytimesn7cgmftshazwhfgzm37qxb44r64ytbb2dj3x62d2lljsciiyd.onion"
*   docker run --rm --net=sswitch alpine/curl --socks5-hostname "socks5:5353" "https://www.nytimesn7cgmftshazwhfgzm37qxb44r64ytbb2dj3x62d2lljsciiyd.onion"