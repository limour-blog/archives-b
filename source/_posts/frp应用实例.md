---
title: FRP应用实例
tags:
  - frp
id: '371'
categories:
  - - Raspberry Pi
date: 2020-08-12 01:26:25
---

参考资料:  
[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)  
[https://github.com/fatedier/frp/blob/master/README\_zh.md](https://github.com/fatedier/frp/blob/master/README_zh.md)  
[https://github.com/fatedier/frp/blob/master/conf/frps\_full.ini](https://github.com/fatedier/frp/blob/master/conf/frps_full.ini)  
[https://github.com/fatedier/frp/blob/master/conf/frpc\_full.ini](https://github.com/fatedier/frp/blob/master/conf/frpc_full.ini)

```
mkdir frp
tar -zxvf frp_0.29.1_linux_amd64.tar.gz -C /root/frp/
```

```
#frps.ini
[common]
tls_only = true
authentication_method = token
token = ****
bind_port = 21000
bind_udp_port = 20999
kcp_bind_port = 21000
dashboard_port = 11750
dashboard_user = Limour
dashboard_pwd = ****
allow_ports = 21001-21999
subdomain_host = frp.limour.top
vhost_http_port = 21080
vhost_https_port = 21443
log_file = /root/frp/frps.log
log_level = info
log_max_days = 3
```

```
# 创建后台启动模版
nano /etc/systemd/system/frp.service

# 内容如下：
[Unit]
Description=frps
After=network.target

[Service]
ExecStart=/root/frp/frp_0.33.0_linux_amd64/frps -c /root/frp/frp_0.33.0_linux_amd64/frps.ini

[Install]
WantedBy=multi-user.target

```

```
#certbot certonly  -d "*.frp.limour.top" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory
```

```
<VirtualHost *:443>
ServerName frp.limour.top
ProxyPreserveHost On
ProxyRequests Off
ProxyPass /frp http://127.0.0.1:11750
ProxyPassReverse /frp http://127.0.0.1:11750

ProxyPass / http://127.0.0.1:11750/
ProxyPassReverse / http://127.0.0.1:11750/
</VirtualHost>

<VirtualHost *:443>
ServerName www.frp.limour.top
ServerAlias *.frp.limour.top
DocumentRoot /var/www/html
SSLEngine on
SSLProtocol all -SSLv2 -SSLV3
SSLCipherSuite HIGH:3DES:!aNULL:!MD5:!SEED:!IDEA
SSLCertificateFile /etc/letsencrypt/live/frp.limour.top/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/frp.limour.top/privkey.pem

# Use RewriteEngine to handle websocket connection upgrades
RewriteEngine On
RewriteCond %{HTTP:Connection} Upgrade [NC]
RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteRule /(.*) ws://127.0.0.1:21080/$1 [P,L]

ProxyPreserveHost On
ProxyRequests Off

```

```
#frpc.ini
[common]
server_addr = frp.limour.top
server_port = 21000
tls_enable = true
token = ****
user = rasp

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 21022

[web01]
type = http
local_ip = 127.0.0.1
local_port = 8888
use_compression = true
subdomain = bt0

```

```
# 创建后台启动模版
sudo nano /etc/systemd/system/frp.service
 
[Unit]
Description=frpc service
After=network.target syslog.target network-online.target
Wants=network.target
Requires=network-online.target

[Service]
Type=simple
ExecStart=/mnt/frp/frp_0.33.0_linux_arm/frpc -c /mnt/frp/frp_0.33.0_linux_arm/frpc.ini
Restart=on-failure
RestartSec=5s
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

```
#测试以下地址
https://frp.limour.top
https://bt0.frp.limour.top
https://bdy0.frp.limour.top
ssh -p 21022 pi@frp.limour.top
```