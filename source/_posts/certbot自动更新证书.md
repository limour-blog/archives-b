---
title: certbot自动更新证书
tags:
  - certbot
id: '471'
categories:
  - - 建站探索
date: 2020-12-12 12:57:13
---

```
#撤销limour.top证书
certbot revoke --cert-path /etc/letsencrypt/live/limour.top/fullchain.pem
```

```
#https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au
git clone https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au certbot-wildcard
cd certbot-wildcard
chmod 0777 au.sh
```

```
#https://ram.console.aliyun.com/users
 创建一个用户，添加云解析管理权限
#用户登录名称 ***@limour.onaliyun.com
#AccessKey ID ****
#AccessKey Secret ****
#将其配置在 au.sh 文件中
```

```
#重新申请证书
certbot certonly  -d limour.top --manual --preferred-challenges dns  --manual-auth-hook "/root/certbot-wildcard/au.sh php aly add" --manual-cleanup-hook "/root/certbot-wildcard/au.sh php aly clean"
```

```
#更新全部证书
certbot renew  --manual --preferred-challenges dns --manual-auth-hook "/root/certbot-wildcard/au.sh php aly add" --manual-cleanup-hook "/root/certbot-wildcard/au.sh php aly clean" --manual-public-ip-logging-ok

```

```
#设置自动更新
nano /root/updatecert.sh

#!/bin/bash
certbot renew  --manual --preferred-challenges dns --manual-auth-hook "/root/certbot-wildcard/au.sh php aly add" --manual-cleanup-hook "/root/certbot-wildcard/au.sh php aly clean"
systemctl restart httpd

chmod 0777 /root/updatecert.sh

```