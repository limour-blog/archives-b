---
title: 树莓派4通过docker安装jupyter
tags:
  - Apache
  - Docker
  - frp
  - jupyter
  - nginx
  - raspberrypi
id: '460'
categories:
  - - Raspberry Pi
date: 2020-12-12 09:09:17
---

```
基础配置参见以下两篇：
https://limour.top/409.html
https://limour.top/453.html
```

```
#到https://hub.docker.com/检索适合arm架构的image
#比如https://hub.docker.com/r/andresvidal/jupyter-armv7l
sudo docker pull andresvidal/jupyter-armv7l
docker run -d \
    --name jupyter \
    --restart unless-stopped \
    -p 3888:8888 \
    -v /mnt/data:/notebooks \
    andresvidal/jupyter-armv7l --NotebookApp.token=''
```

```
#在树莓派4的宝塔面板中nginx配置中http段添加以下内容
#同时开放3888和3880端口

# top-level http config for websocket headers
# If Upgrade is defined, Connection = upgrade
# If Upgrade is empty, Connection = close
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
    listen 3880;
    server_name jupyter4;
    # Managing literal requests to the JupyterHub front end
    location / {
        proxy_pass http://127.0.0.1:3888;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # websocket headers
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header X-Scheme $scheme;

```

```
#在树莓派4的宝塔面板文件中，修改frpc.ini
#末尾添加以下内容
[web03]
type = http
local_ip = 127.0.0.1
local_port = 3880
use_compression = true
http_user = Limour
http_pwd = ***
subdomain = jupyter4

```

```
#到frp服务器上修改apache配置，添加以下内容
#添加位置参见https://limour.top/371.html

nano /etc/httpd/conf.d/zproxy.conf
# Use RewriteEngine to handle websocket connection upgrades
RewriteEngine On
RewriteCond %{HTTP:Connection} Upgrade [NC]
RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteRule /(.*) ws://127.0.0.1:21080/$1 [P,L]

```

[![](https://img-cdn.limour.top/blog_wp/2020/12/image.png)](https://img-cdn.limour.top/blog_wp/2020/12/image.png)

最终效果

[![](https://img-cdn.limour.top/blog_wp/2020/12/image-1.png)](https://img-cdn.limour.top/blog_wp/2020/12/image-1.png)

容器细节