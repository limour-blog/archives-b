---
title: 【从零开始配置VPS】05：Lsky Pro + WebP-SH
tags: []
id: '2105'
categories:
  - - uncategorized
date: 2023-01-05 05:10:31
---

## 第零步 迁出OSS的图片数据

*   wget https://gosspublic.alicdn.com/ossfs/ossfs\_1.80.7\_ubuntu20.04\_amd64.deb

*   sudo apt-get install gdebi-core

*   sudo gdebi ossfs\_1.80.7\_ubuntu20.04\_amd64.deb

*   echo BucketName:yourAccessKeyId:yourAccessKeySecret > /etc/passwd-ossfs

*   chmod 640 /etc/passwd-ossfs

*   mkdir img

*   ossfs BucketName img -o url=http://oss-cn-shanghai-internal.aliyuncs.com

*   tar -czvf img.tar.gz img

## 第一步 搭建Lsky Pro

```yml
version: '3.3'
services:
    lsky-pro:
        container_name: lsky-pro
        restart: always
        volumes:
            - './lsky-pro-data:/var/www/html'
        ports:
            - '7791:80'
        image: 'dko0/lsky-pro'
```

*   mkdir -p ~/app/Lsky && cd ~/app/Lsky && nano docker-compose.yml

*   docker-compose up -d

*   反向代理 7791 端口

*   修改 【储存策略】【默认本地策略】【访问网址】为 `https://img-cdn.limour.top/i`

*   `sudo chmod 777 -R /root/app/Lsky/lsky-pro-data/storage/app/uploads`

![](https://img-cdn.limour.top/i/2023/01/05/63b65a696e2af.png)

### 第零步 导入迁出的图片数据

*   tar -xzvf img.tar.gz

*   mv img/\* /root/app/Lsky/lsky-pro-data/storage/app/uploads

*   rm -r img img.tar.gz

## 第二步 搭建WebP-SH

```yml
version: '3'

```

*   mkdir -p ~/app/WebP && cd ~/app/WebP && nano docker-compose.yml

*   docker-compose up -d

*   反向代理 3333 端口 到 `https://img-cdn.limour.top/`

## 第三步 搭建随机图片API

```php
<?php
$arr=file('photos');
$n=count($arr)-1;
$x=rand(0,$n);
header("Location:".$arr[$x],"\n");
header("Cache-Control:no-store, max-age=0");
?> 
```

*   cd ~/app/WordPress/www

*   新建 photos 文本文件，一行一个图片地址

*   nano r.php

*   将 Sakurairo 主题 的 【主页设置】【封面设置】的随机图片地址换成 `https://limour.top/r.php`