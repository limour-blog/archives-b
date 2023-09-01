---
title: 通过VMware构建本博客使用的生信处理环境
tags: []
id: '1944'
categories:
  - - uncategorized
date: 2022-07-31 17:30:20
---

使用Windows系统，安装Docker需要开启WSL2，可能存在未知的BUG，操作和维护也不是那么轻松，因此本文将教大家如何使用VMware安装Ubuntu，再通过Ubuntu安装Docker，然后通过Docker安装Jupyter Docker Stacks

## 第一步 安装VMware

复旦大学是VMware学术计划成员，因此在校师生可以在遵循相关协议的前提下免费使用VMware学术计划内所包含产品。[点此按信息办的步骤即可完成VMware的安装](http://www.ecampus.fudan.edu.cn/2270/list.htm)

## 第二步 安装Ubuntu

从[Ubuntu官网](https://cn.ubuntu.com/download/server/thank-you?version=22.04&architecture=amd64)下载ISO镜像，创建虚拟机

![](https://img-cdn.limour.top/2022/08/01/62e6aa4038aa6.png)

选择典型

![](https://img-cdn.limour.top/2022/08/01/62e6aa768774c.png)

选择ISO镜像路径

![](https://img-cdn.limour.top/2022/08/01/62e6aaea6e083.png)

选择虚拟磁盘保存路径，建议新建空文件夹

![](https://img-cdn.limour.top/2022/08/01/62e6abbfa6a51.png)

磁盘大小设置大一点

![](https://img-cdn.limour.top/2022/08/01/62e6add58a37b.png)

一路默认，到这里选择开启SSH，安装完成后会提示reboot，之后会提示移除CD安装镜像

*   重启后输入账号密码登录
*   sudo apt install net-tools
*   sudo ifconfig 记录下服务器IP
*   使用FinalShell等SSH工具进行连接

## 第三步 Ubuntu换源

```ini
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
```

*   sudo -i
*   nano /etc/apt/sources.list
*   apt update

## 第四步 安装docker

```json
{
    "registry-mirrors":[
                        "https://hub-mirror.c.163.com/",
                        "https://docker.mirrors.ustc.edu.cn/"
                        ]
}
```

*   `curl -sSL https://get.daocloud.io/docker sh`
*   `curl -L https://get.daocloud.io/docker/compose/releases/download/v2.8.0/docker-compose-$(uname -s)-$(uname -m) > /usr/local/bin/docker-compose`
*   `chmod +x /usr/local/bin/docker-compose`
*   `docker -v`
*   `docker-compose -v`
*   nano /etc/docker/daemon.json
*   systemctl daemon-reload
*   systemctl restart docker

## 第五步 磁盘扩容

*   vgdisplay
*   lvresize -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
*   resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

## 第六步 安装Jupyter Docker Stacks

*   [安装R内核的Jupyter](https://occdn.limour.top/1530.html)
*   [启用Jupyter的代码提示功能](https://occdn.limour.top/1532.html)