---
title: 利用Rclone的copy命令转存云盘文件夹
tags: []
id: '1905'
categories:
  - - uncategorized
date: 2022-07-14 00:44:10
---

[七米蓝的仓库](https://al.chirmyram.com/rep/Music)上有挺多歌曲，我想转存到自己的ondrive上，这时候可以使用Rclone的copy命令来自动完成文件夹的转存了。

## 挂载七米蓝的仓库

参数

值

链接 / URL

https://al.chirmyram.com/dav/

主机 / Host

al.chirmyram.com

路径 / Path

/dav/

协议 / HTTPS

SSL

端口 / Port

443

账号 / User

alist

密码 / Password

alist

七米蓝的仓库支持webdav的方式挂载，参数如上，我们利用rclone来挂载它

*   sudo -i
*   rclone config
*   n
*   chirmyram
*   45 WebDav
*   https://al.chirmyram.com/dav/
*   5
*   alist
*   y
*   alist
*   enter
*   enter
*   y
*   q

## 找到要转存的目录

*   rclone lsd chirmyram:
*   rclone lsd chirmyram:rep
*   rclone lsd chirmyram:rep/Music
*   rclone lsd onedrive:hk
*   rclone lsd onedrive:hk/Music
*   rclone lsd onedrive:hk/Music/chirmyram

## 进行转存

*   rclone copy --ignore-existing --progress --ignore-errors chirmyram:rep/Music onedrive:hk/Music/chirmyram