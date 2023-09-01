---
title: Jupyter添加Golang内核
tags: []
id: '2040'
categories:
  - - uncategorized
date: 2022-10-02 16:01:19
---

*   conda create -n golang -c conda-forge go -y
*   conda activate golang
*   go env -w GO111MODULE=on
*   go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/
*   go install github.com/gopherdata/gophernotes@v0.7.5 #去仓库查看最新版本号
*   mkdir -p ~/.local/share/jupyter/kernels/golang
*   cd ~/.local/share/jupyter/kernels/golang
*   cp "$(go env GOPATH)"/pkg/mod/github.com/gopherdata/gophernotes@v0.7.5/kernel/\* "."
*   chmod +w ./kernel.json
*   sed "sgophernotes$(go env GOPATH)/bin/gophernotes" < kernel.json.in > kernel.json