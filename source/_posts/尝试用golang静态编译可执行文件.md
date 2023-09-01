---
title: 尝试用Golang静态编译可执行文件
tags: []
id: '2150'
categories:
  - - uncategorized
date: 2023-03-01 15:38:16
---

用alpine制作docker镜像，需要静态编译的可执行文件

准备工作：[Jupyter添加Golang内核](https://occdn.limour.top/2381.html)

### 安装musl-gcc

*   sudo docker exec -u 0 -it jupyterR bash
*   apt update
*   apt-get install musl-dev musl-tools
*   exit

### 更新golang

*   conda activate golang
*   go install golang.org/dl/go1.20.1@latest
*   /home/jovyan/go/bin/go1.20.1 download
*   /home/jovyan/go/bin/go1.20.1 version

### 静态编译

*   cd github\_repo/
*   git clone --depth=1 https://ghproxy.com/https://github.com/XTLS/Xray-core.git
*   cd Xray-core/
*   CC=musl-gcc /home/jovyan/go/bin/go1.20.1 build -tags musl -o xray -trimpath -ldflags '-linkmode "external" -extldflags "-static" -s -w -buildid=' ./main
*   ldd xray
*   ./xray -version