---
title: 【从零开始配置VPS】02：服务配置
tags: []
id: '2097'
categories:
  - - uncategorized
date: 2023-01-03 10:08:17
---

## 第一步 安装 Nginx Proxy Manager

```yml
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

*   mkdir -p ~/base/NGPM && cd ~/base/NGPM

*   nano docker-compose.yml

*   sudo docker-compose up -d

*   登录到 http://ip:81

*   Email: `admin@example.com`

*   Password: `changeme`

*   反代 Nginx Proxy Manager

*   ip addr show docker0

## 第二步 安装frp

```yml
version: '3.3'
services:
    frps:
        restart: always
        network_mode: host
        volumes:
            - './frps.ini:/etc/frp/frps.ini'
            - './frps_log:/tmp/frps_log'
        container_name: frps
        image: snowdreamtech/frps
```

```ini
[common]
tls_only = true
authentication_method = token
token = <token>
bind_port = 21000
bind_udp_port = 20999
kcp_bind_port = 21000
dashboard_port = 11750
dashboard_user = Limour
dashboard_pwd = <passwd>
allow_ports = 21001-21999
subdomain_host = limour.top
vhost_http_port = 21080
vhost_https_port = 21443
 
log_file = /tmp/frps_log/frps.log
log_level = info
log_max_days = 3
```

*   mkdir -p ~/base/FRP && cd ~/base/FRP

*   nano docker-compose.yml

*   nano frps.ini

*   sudo docker-compose up -d

*   tail frps\_log/frps.log

*   反代 dashboard 11750

## 第三步 安装nps

```yml
version: '3.3'
services:
    nps:
        container_name: nps
        restart: unless-stopped
        network_mode: host
        volumes:
            - './conf:/conf'
            - '/etc/localtime:/etc/localtime:ro'
        image: yisier1/nps
```

```conf
appname = nps
#Boot mode(devpro)
runmode = pro
 
#HTTP(S) proxy port, no startup if empty
http_proxy_ip=0.0.0.0
http_proxy_port=18081
 
##bridge
bridge_type=tcp
bridge_port=8024
bridge_ip=0.0.0.0
 
#Traffic data persistence interval(minute)
#Ignorance means no persistence
#flow_store_interval=1
 
# log level LevelEmergency->0  LevelAlert->1 LevelCritical->2 LevelError->3 LevelWarning->4 LevelNotice->5 LevelInformational->6 LevelDebug->7
log_level=7
#log_path=nps.log
 
#Whether to restrict IP access, true or false or ignore
#ip_limit=true
 
#allow_ports=9001-9009,10001,11000-12000
 
#Web management multi-user login
allow_user_login=false
allow_user_register=false
allow_user_change_username=false
 
#extension
allow_flow_limit=false
allow_rate_limit=false
allow_tunnel_num_limit=false
allow_local_proxy=false
allow_connection_num_limit=false
allow_multi_ip=false
system_info_display=true
 
#cache
http_cache=false
http_cache_length=100
 
#get origin ip
http_add_origin_header=false
 
#pprof debug options
#pprof_ip=0.0.0.0
#pprof_port=9999
 
#client disconnect timeout
disconnect_timeout=60
 
# 以下的需要进行配置
# Public password, which clients can use to connect to the server
# After the connection, the server will be able to open relevant ports and parse related domain names according to its own configuration file.
public_vkey=<16个字符>
 
#Web API unauthenticated IP address(the len of auth_crypt_key must be 16)
#Remove comments if needed
auth_key=<24个字符>
auth_crypt_key=<16个字符>
 
#web
web_host=limour.top
web_username=Limour
web_password=<16个字符>
web_port = 8080
web_ip=0.0.0.0
web_open_ssl=false
web_base_url=
open_captcha=true
# if web under proxy use sub path. like http://host/nps need this.
#web_base_url=/nps
 
#p2p
p2p_ip=<写服务器的ip>
p2p_port=6000
# 设置为6000，请在控制台防火墙开放6000~6002(额外添加2个端口)udp端口
```

*   mkdir -p ~/base/NPS && cd ~/base/NPS && mkdir conf

*   nano docker-compose.yml

*   nano conf/nps.conf

*   touch conf/{clients,hosts,tasks}.json

*   sudo docker-compose up -d

*   反代 dashboard 8080