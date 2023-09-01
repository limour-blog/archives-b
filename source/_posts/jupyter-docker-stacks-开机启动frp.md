---
title: Jupyter Docker Stacks 开机启动frp
tags: []
id: '1387'
categories:
  - - 超算基础
date: 2022-01-15 12:44:54
---

## 第一步 配置frp启动脚本

```shell
#!/usr/bin/sh
HOME=/home/jovyan
nohup $HOME/dev/frp_0.38.0_linux_amd64/frpc -c $HOME/etc/frpc.ini > $HOME/log/frp.log 2>&1 &
```

## 第二步 配置docker启动脚本

```shell
#!/bin/sh
docker run -d -p 57002:8888 --name jupyterrf \
--restart always \
-m 60416M --memory-swap -1 \
-c 1024 \
--cpus 16 \
-v /home/gene:/home/jovyan/gene \
-v /home/gene/zl_liu/jupytera/opt:/opt \
-v /home/gene/zl_liu/jupytera/before-notebook.d:/usr/local/bin/before-notebook.d \
-v /home/gene/zl_liu/jupytera:/home/jovyan jupyter/datascience-notebook:r-4.1.1 \
start-notebook.sh --NotebookApp.token='yourtoken'
```

## 第三步 完成配置并启动

*   将frp启动脚本移动到 `/home/gene/zl_liu/jupytera/before-notebook.d`
*   运行docker启动脚本