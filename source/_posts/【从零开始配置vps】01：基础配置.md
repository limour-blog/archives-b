---
title: 【从零开始配置VPS】01：基础配置
tags: []
id: '2096'
categories:
  - - uncategorized
date: 2023-01-03 08:43:04
---

## 第一步 添加SWAP

*   wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh && chmod +x box.sh && clear && sudo ./box.sh

*   18 大小输入4096，设置4G大小的swap空间

## 第二步 开启BBR

*   wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh && chmod +x box.sh && clear && sudo ./box.sh

*   17 11 开启BBR-FQ

*   sudo reboot

## 第三步 修改时区

*   timedatectl list-timezones

*   sudo timedatectl set-timezone Asia/Shanghai

## 第四步 修改SSH端口

*   sudonano /etc/ssh/sshd\_config

*   修改 Port 22 为 Port 2022

*   sudo service sshd restart

*   新开一个SSH连接测试修改是否成功

## 第五步 安装docker

*   sudo apt update

*   sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

*   curl -fsSL https://download.docker.com/linux/ubuntu/gpg sudo apt-key add -

*   sudo add-apt-repository "deb \[arch=amd64\] https://download.docker.com/linux/ubuntu $(lsb\_release -cs) stable"

*   sudo apt update

*   sudo apt install docker-ce

*   sudo systemctl status docker

*   sudo curl -L "https://github.com/docker/compose/releases/download/v2.14.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

*   sudo chmod +x /usr/local/bin/docker-compose

*   docker-compose --version