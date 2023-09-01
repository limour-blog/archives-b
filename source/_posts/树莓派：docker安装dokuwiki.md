---
title: 树莓派：Docker安装dokuwiki
tags: []
id: '1889'
categories:
  - - uncategorized
date: 2022-07-11 19:33:15
---

## 来源

[https://blog.csdn.net/baozi\_xiaoge/article/details/103642758](https://blog.csdn.net/baozi_xiaoge/article/details/103642758)

[https://hub.docker.com/r/linuxserver/dokuwiki](https://hub.docker.com/r/linuxserver/dokuwiki)

[https://www.jianshu.com/p/729d755ace65](https://www.jianshu.com/p/729d755ace65)

[https://www.jianshu.com/p/80a9308e9586](https://www.jianshu.com/p/80a9308e9586)

[https://www.jianshu.com/p/d986ec44d5f3](https://www.jianshu.com/p/d986ec44d5f3)

## 部署

```yml
version: "2.1"
services:
  dokuwiki:
    image: lscr.io/linuxserver/dokuwiki:latest
    container_name: dokuwiki
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./config:/config
    ports:
      - 8088:80
    restart: unless-stopped
```

*   sudo lsof -i:8088
*   mkdir dokuwiki && cd dokuwiki
*   nano docker-compose.yml
*   找一台外网的机器
*   wget https://raw.githubusercontent.com/NotGlop/docker-drag/master/docker\_pull.py
*   python3 docker\_pull.py lscr.io/linuxserver/dokuwiki@sha256:05996b6c3e9d62c9a5c10a126051643b5af9178184c04b38e4b2903a3f251980
*   gzip linuxserver\_dokuwiki.tar
*   sudo apt install rsync
*   rsync -a -P -e "ssh -p 2222" ./linuxserver\_dokuwiki.tar.gz pi@limour.top:/tmp/
*   回到树莓派上
*   gunzip linuxserver\_dokuwiki.tar.gz
*   sudo docker load < linuxserver\_dokuwiki.tar
*   查看镜像 sudo docker images
*   修改标签 sudo docker tag 4e0459e9148f lscr.io/linuxserver/dokuwiki:latest
*   cd /home/pi/dokuwiki
*   测试一下镜像正确：sudo docker pull lscr.io/linuxserver/dokuwiki:latest
*   sudo docker-compose up -d
*   访问 http://IP-ADDRESS:PORT/install.php 完成安装
*   修复中文乱码：编辑 dokuwiki/conf/local.php 文件
*   添加：$conf\['fnencode'\] = 'utf-8';

## 穿透

```ini
[dokuwiki]
type = http
local_ip = 127.0.0.1
local_port = 8088
use_compression = true
subdomain = wiki
```

## 演示

[https://j11.fun/wiki](https://j11.fun/wiki)

## 更换主题

*   中文模板列表：[https://www.dokuwiki.org/zh:template](https://www.dokuwiki.org/zh:template)
*   下载zip文件后，在管理-拓展安装-手动安装里上传安装

## 添加插件

*   插件列表：https://www.dokuwiki.org/plugins
*   新页面插件：Add New Page Plugin
*   MD插件：Markdown Page Plugin
*   移动目录插件：Move Plugin
*   导航菜单插件：IndexMenu Plugin

## 集成中文分词（编译出错，项目停止了）

*   sudo docker exec -it dokuwiki /bin/bash
*   cd /tmp
*   php -v 查看php版本 7.4.26
*   宿主机
*   sudo apt install -y libxml2 libxml2-dev libjpeg-dev libcurl4-openssl-dev libreadline-dev libzip-dev libfreetype6-dev libssl-dev libsqlite3-dev libonig-dev
*   cd /tmp
*   proxychains4 wget https://www.php.net/distributions/php-7.4.26.tar.gz
*   tar -zxvf php-7.4.26.tar.gz
*   cd php-7.4.26
*   sudo apt install gcc g++
*   ./configure --prefix=/usr/local/dokuwiki/php7
*   make && sudo make install
*   cd /tmp
*   wget -q -O - http://www.xunsearch.com/scws/down/scws-1.2.1.tar.bz2 tar xjf -
*   cd scws-1.2.1
*   ./configure --prefix=/usr/local/dokuwiki/scws --build=aarch64-unknown-linux-gnu
*   sudo make install
*   cd phpext/
*   sudo apt install autoconf
*   /usr/local/dokuwiki/php7/bin/phpize
*   ./configure --prefix=/usr/local/dokuwiki/scwsphp --with-scws=/usr/local/dokuwiki/scws --with-php-config=/usr/local/dokuwiki/php7/bin/php-config
*   **编译出错，放弃**