---
title: Android平板利用Termux做SSH客户端
tags: []
id: '2161'
categories:
  - - uncategorized
date: 2023-03-13 06:41:17
---

## 安装Termux

*   下载 **[termux-app](https://github.com/termux/termux-app)**（点此[镜像加速](https://occdn.limour.top/2561.html)）

*   pkg install proot-distro

*   proot-distro install alpine

*   proot-distro login alpine

*   `sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories`

*   apk update && apk add openssh

## 配置快捷登录

```ini
host SER5
  user root
  hostname limour.top
  Port 22
  # PreferredAuthentications publickey
  # IdentityFile ~/.ssh/id_rsa
```

*   nano ~/.ssh/config

*   ssh-keygen -t rsa

*   ssh-copy-id SER5

*   nano ~/.ssh/config 将注释取消掉

*   ssh SER5 即可直接登录