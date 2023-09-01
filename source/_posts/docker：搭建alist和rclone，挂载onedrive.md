---
title: Docker：搭建AList和Rclone，挂载OneDrive
tags: []
id: '1901'
categories:
  - - uncategorized
date: 2022-07-12 00:17:59
---

淘宝上了一辆6人家庭共享office365的车，来试试1T的OneDrive空间好不好用😆

## AList

```yml
version: '3.3'
services:
    alist:
        restart: always
        volumes:
            - './alist:/opt/alist/data'
        ports:
            - '5244:5244'
        container_name: alist
        image: 'xhofe/alist:latest'
```

*   mkdir alist && cd alist
*   nano docker-compose.yml
*   docker-compose up -d
*   反代 5244
*   默认的密码在 docker-compose logs 里
*   挂载OneDrive时数据不会及时同步，更新需后台登录之后返回列表，然后到改动的目录右键刷新，或者等程序缓存一小时自动失效

参考1：咕咕鸽 https://blog.laoda.de/archives/docker-install-alist  
参考2：作者文档 https://alist-doc.nn.ci/docs/driver/onedrive  
参考3：**[srcenchen](https://github.com/srcenchen)**的issue https://github.com/alist-org/alist/issues/770

## Rclone

```ini
[Unit]
Description=rclone
 
[Service]
User=root
ExecStart=/usr/bin/rclone mount onedrive: /root/rclone --allow-other --allow-non-empty --vfs-cache-mode writes
Restart=on-abort
 
[Install]
WantedBy=multi-user.target
```

*   **[https://rclone.org/downloads/](https://rclone.org/downloads/)** 下载电脑版，解压，进入目录，左上角文件，打开PowerShell
*   ./rclone config
*   n
*   onedrive
*   32 MS OneDrie
*   enter
*   enter
*   1
*   enter
*   enter
*   1
*   enter
*   enter
*   复制下窗口的json
*   enter
*   q
*   登录服务器
*   mkdir rclone && cd rclone
*   curl https://rclone.org/install.sh sudo bash
*   rclone config
*   n
*   onedrive
*   32 MS OneDrie
*   enter
*   enter
*   1
*   enter
*   n
*   粘贴token
*   1
*   enter
*   enter
*   q
*   apt install fuse
*   rclone mount onedrive: /root/rclone --allow-other --allow-non-empty --vfs-cache-mode writes
*   Ctrl+C 取消
*   whereis rclone # /usr/bin/rclone
*   nano /usr/lib/systemd/system/rclone.service
*   systemctl daemon-reload
*   systemctl start rclone
*   systemctl enable rclone
*   systemctl status rclone

参考1：咕咕鸽 https://blog.laoda.de/archives/aria2-rclone-filebrowser  
参考2：咕咕鸽 https://blog.laoda.de/archives/make-vps-bigger