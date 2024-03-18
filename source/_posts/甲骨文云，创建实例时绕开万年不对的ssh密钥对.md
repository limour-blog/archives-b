---
title: 甲骨文云，创建实例时绕开万年不对的SSH密钥对
tags: []
id: '2115'
categories:
  - - uncategorized
date: 2023-01-29 17:25:55
---

创建新的免费实例，选择Ubuntu镜像，SSH密钥对在登录时一直不正确，删除重建了N次，干脆绕开这个奇怪的设计。按下图在最后的高级选项里添加如下的初始化脚本，然后用密码登录即可。

![](https://img.limour.top/archives_2023/2023/01/30/63d6ab6e72577.webp)

```bash
#!/bin/bash
echo root:123456789 sudo chpasswd root
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config;
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;
sudo service sshd restart
```