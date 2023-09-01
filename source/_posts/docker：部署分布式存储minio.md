---
title: Docker：部署分布式存储minIO
tags: []
id: '1910'
categories:
  - - uncategorized
date: 2022-07-16 04:43:20
---

## 准备

*   甲骨文免费VPS一台，解析到minio1.j11.fun，开放9000、9001
*   阿里云香港VPS一台，解析到minio2.j11.fun，开放9000、9001
*   iptables和防火墙放行对方ip和自己ip：/usr/sbin/iptables -I INPUT -s 0.0.0.0 -j ACCEPT

## 证书

```ini
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = VA
L = Somewhere
O = MyOrg
OU = MyOU
CN = minio2.j11.fun

[v3_req]
subjectAltName = @alt_names

```

*   mkdir -p ~/minio/config/CAs && cd ~/minio/config
*   nano openssl.conf
*   openssl req -new -x509 -nodes -days 730 -keyout private.key -out public.crt -config openssl.conf
*   将各个服务器的crt放到自己的CAs目录下

## 部署

```yml
version: '3.7'
services:
  minio:
    restart: unless-stopped
    hostname: minio2.j11.fun
    image: minio/minio
    volumes:
      - ./data1:/data1
      - ./data2:/data2
      - ./config:/root/.minio/certs
    network_mode: host
    extra_hosts:
      - "minio1.j11.fun:155.248.208.5"  # 1st node
      - "minio2.j11.fun:127.0.0.1"  # 2nd node
    environment:
      MINIO_ROOT_USER: ***
      MINIO_ROOT_PASSWORD: ***
    command: server --console-address ":9001" https://minio{1...2}.j11.fun/data{1...2}
```

*   cd ~/minio
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   sudo docker-compose logs
*   sudo docker-compose down && sudo docker volume prune
*   echo "127.0.0.1 minio2.j11.fun" >> /etc/hosts
*   反代任意一台的9001端口，协议https

## 效果

[https://minio.j11.fun/](https://minio.j11.fun/)

![](https://img-cdn.limour.top/blog/20220716124213.png)