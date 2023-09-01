---
title: FTP客户端使用记录
tags: []
id: '2079'
categories:
  - - uncategorized
date: 2022-11-12 10:04:50
---

*   conda install -c conda-forge lftp -y

*   lftp ftp://用户名\[:密码\]@服务器地址\[:端口\]

*   set pget:default-n 5 # 设置 **pget** 使用的线程数

*   set ftp:charset gbk #设置远程编码为gbk

*   set file:charset utf8 #设置本地编码为utf8

*   set xfer:clobber on #允许覆盖

*   help

命令

本地

远程

显示工作目录

lpwd

pwd

切换目录

lcd

cd

显示文件列表

!ls

cls

显示文件列表

!ls -l

ls

统计文件大小

du

移动、重命名

mv

删除

rm

创建文件夹

mkdir

删除文件夹

rmdir

**下载**

**上传**

单个文件

get

put

多个文件

mget

mput

多线程

pget

目录

mirror

mirror -R

*   断点续传 添加 -c 参数

*   上传重命名 put -c local\_file -o myfile