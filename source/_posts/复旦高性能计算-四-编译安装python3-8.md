---
title: 复旦高性能计算 (四) 编译安装python3.8
tags: []
id: '724'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-09-21 20:04:30
---

## 第一步 安装依赖

*   编译安装sqlite3
*   下载源文件：[SQLite Download Page](https://www.sqlite.org/download.html)

```
wget https://www.sqlite.org/snapshot/sqlite-snapshot-202108231028.tar.gz
tar -zxvf sqlite-snapshot-202108231028.tar.gz 
./configure --prefix=/home/rqzhang/zlliu/dev/sqlite3
make -j8 && make install
ln -s /home/rqzhang/zlliu/dev/sqlite3/bin/* /home/rqzhang/zlliu/bin/
ln -s /home/rqzhang/zlliu/dev/sqlite3/lib/* /home/rqzhang/zlliu/lib/
ln -s /home/rqzhang/zlliu/dev/sqlite3/include/* /home/rqzhang/zlliu/include/
```

## 第二步 下载源码包并解压

```
wget https://www.python.org/ftp/python/3.8.9/Python-3.8.9.tar.xz
tar -Jxvf Python-3.8.9.tar.xz
cd Python-3.8.9
./configure --prefix=/home/rqzhang/zlliu/dev/python38 --enable-optimizations --enable-loadable-sqlite-extensions --enable-shared 
```

## 第三步 编译安装并链接

*   编译安装python3.8

```
make -j8 && make install
ln -s /home/rqzhang/zlliu/dev/python38/bin/python3.8 /home/rqzhang/zlliu/bin/
ln -s /home/rqzhang/zlliu/dev/python38/bin/pip3.8 /home/rqzhang/zlliu/bin/
ln -s /home/rqzhang/zlliu/dev/python38/bin/easy_install-3.8 /home/rqzhang/zlliu/bin/
ln -s /home/rqzhang/zlliu/dev/python38/include/* /home/rqzhang/zlliu/include/
ln -s /home/rqzhang/zlliu/dev/python38/lib/* /home/rqzhang/zlliu/lib/
```

## 第四步 添加到jupyter内核

```
pip3.8 install ipykernel
python3.8 -m ipykernel install --user --name python3.8 --display-name "Python3.8"
```