---
title: acme 自动更新证书
tags: []
id: '1140'
categories:
  - - 建站探索
  - - 计算机相关
date: 2021-10-25 01:39:29
---

## 第一步 取消 [certbot](https://limour.top/471.html)

```
nano /etc/crontab
# 删除 0 1 9 * * root /root/updatecert.sh > /root/upclog.txt
```

```
export EDITOR="/usr/bin/nano"
crontab -l
crontab -e
```

## 第二步 安装 acme.sh

*   安装 proxychans4

```
# https://github.com/rofl0r/proxychains-ng/releases 上传源码
tar -zxvf proxychains-ng-4.15.tar.gz 
cd proxychains-ng-4.15
yum groupinstall "Development Tools" "Development Libraries" # apt install build-essential 
./configure --prefix=/usr --sysconfdir=/etc
make && make install
make install-config
nano -K /etc/proxychains.conf
# socks5 127.0.0.1 20808
```

*   安装 xray

```
# https://github.com/XTLS/Xray-core/releases 上传 Xray-linux-64.zip
unzip Xray-linux-64.zip
```

*   开启 xray

```
# 上传xray配置文件
# 修改格式
vi ./xui2.json
:set ff
:set ff=unix
:wq
# 运行
./xray run -c ./xui2.json &
# 查看
jobs
```

*   安装 acme.sh

**proxychains 只会代理 TCP 连接，而 ping 使用的是 ICMP。记住这一点即可。**

```
proxychains4 bash
curl https://get.acme.sh  sh
```

## 第三步 停止代理

```
jobs
fg
^C
```

## 第四步 查看cron

```
crontab -l
51 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

## 第五步 新建脚本

```
nano -K updatecert.sh

#!/bin/bash
export LE_WORKING_DIR="/root/.acme.sh"
export CF_Key=""
export CF_Email=""
"/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

## 第六步 修改cron

```
crontab -e
51 0 * * * /root/updatecert.sh > /dev/null
```

## 第七步 创建证书

*   吊销原证书

```
certbot revoke --cert-path /etc/letsencrypt/live/limour.top/fullchain.pem
acme.sh --revoke -d prcdn.limour.top
```

*   注册账号
*   申请证书 [Server](https://github.com/acmesh-official/acme.sh/wiki/Server)

```
export LE_WORKING_DIR="/root/.acme.sh"
export CF_Key=""
export CF_Email=""
alias acme.sh="/root/.acme.sh/acme.sh"
acme.sh --register-account -m limour@limour.top
acme.sh --issue --dns dns_cf -d *.limour.top -d limour.top -d *.frp.limour.top --server https://acme-v02.api.letsencrypt.org/directory
```

## 第八步 配置证书

*   修改httpd配置

```
SSLCertificateFile /etc/letsencrypt/live/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/privkey.pem
```

*   安装证书

```
acme.sh --install-cert -d *.limour.top \
--key-file       /etc/letsencrypt/live/privkey.pem  \
--fullchain-file /etc/letsencrypt/live/fullchain.pem \
--reloadcmd     "systemctl restart httpd"
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-3.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-3.webp)

成功！