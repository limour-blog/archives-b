---
title: Docker：安装nps服务端与客户端
tags: []
id: '1885'
categories:
  - - uncategorized
date: 2022-07-11 06:12:45
---

## 来源

[https://github.com/ehang-io/nps](https://github.com/ehang-io/nps)

[https://ehang-io.github.io/nps/#/install](https://ehang-io.github.io/nps/#/install)

[https://wangchujiang.com/docker-tutorial/nps/index.html](https://wangchujiang.com/docker-tutorial/nps/index.html)

## 服务端

```yml
version: '3.3'
services:
    nps:
        container_name: nps
        restart: unless-stopped
        network_mode: host
        volumes:
            - './conf:/conf'
        image: ffdfgdfg/nps
```

*   mkdir nps && cd nps && mkdir conf
*   nano docker-compose.yml
*   nano conf/nps.conf
*   touch conf/{clients,hosts,tasks}.json
*   sudo docker-compose up -d
*   sudo docker-compose logs
*   lsof -i:8080
*   关闭：sudo docker-compose down
*   在NPM面板中添加8080的反向代理

```ini
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
 
# Public password, which clients can use to connect to the server
# After the connection, the server will be able to open relevant ports and parse related domain names according to its own configuration file.
public_vkey=123
 
#Traffic data persistence interval(minute)
#Ignorance means no persistence
#flow_store_interval=1
 
# log level LevelEmergency->0  LevelAlert->1 LevelCritical->2 LevelError->3 LevelWarning->4 LevelNotice->5 LevelInformational->6 LevelDebug->7
log_level=7
#log_path=nps.log
 
#Whether to restrict IP access, true or false or ignore
#ip_limit=true
 
#p2p
p2p_ip=127.0.0.1
p2p_port=6000
 
#web
web_host=nps.j11.fun
web_username=admin
web_password=123
web_port = 8080
web_ip=0.0.0.0
web_open_ssl=false
web_base_url=
# if web under proxy use sub path. like http://host/nps need this.
#web_base_url=/nps
 
#Web API unauthenticated IP address(the len of auth_crypt_key must be 16)
#Remove comments if needed
#auth_key=test
auth_crypt_key=1234567812345678
 
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
```

*   p2p\_ip 写服务器ip
*   p2p\_port 设置为6000，请在控制台防火墙开放6000~6002(额外添加2个端口)udp端口
*   public\_vkey web\_password auth\_crypt\_key 三处自行设置
*   控制台防火墙开放8024的桥接端口

## 客户端

*   nps web管理-客户端，新建一个客户端，记录下**唯一验证密钥**
*   无配置文件：docker run -d --name npc --net=host ffdfgdfg/npc -server=<ip:port> -vkey=<web界面中显示的密钥>
*   或者使用下面的docker-compose.yml
*   mkdir npc && cd npc
*   nano docker-compose.yml
*   sudo docker-compose up -d
*   sudo docker-compose logs

```yml
version: '3.3'
services:
    npc:
        container_name: npc
        network_mode: host
        image: ffdfgdfg/npc
        restart: unless-stopped
        command: -server=47.242.159.58:8024 -vkey=<web界面中显示的密钥>
```

## 客户端轻量化部署

*   arch 查看CPU架构，下载对应的客户端
*   mkdir npc && cd npc
*   sudo ./npc install -server=ip:port -vkey=web界面中显示的密钥
*   sudo npc start
*   卸载
*   sudo npc stop
*   sudo ./npc uninstall

## 其他方式

*   sudo nano /etc/rc.local
*   /usr/bin/nohup /home/ubuntu/npc\_dc/npc -server=<ip:port> -vkey=<vkey> > /dev/null 2>&1 &