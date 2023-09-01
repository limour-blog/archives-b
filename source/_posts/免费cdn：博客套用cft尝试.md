---
title: 免费CDN：博客套用CFT尝试
tags: []
id: '1948'
categories:
  - - uncategorized
date: 2022-08-05 10:17:19
---

这几天有大佬研究出来薅AWS免费CDN每月1T流量的教程。CF过于良心，高峰时段博客访问太慢，因此尝试一下CFT。

## 第一步 注册AWS

AWS注册比甲骨文云轻松，但是需要VISA信用卡，邮箱和手机号

## 第二步 建立测试网站

*   NPM面板反代wordpress端口，选一个随机子域名进行DNS解析
*   [安装X面板](https://occdn.limour.top/2001.html)，**ws连接中千万不要开启acceptProxyProtocol**
*   NPM面板中按下面的图添加**location**反代
*   [grpc看这里，但是CFT似乎还不支持](https://occdn.limour.top/2075.html)
*   查看日志 tail /home/ubuntu/ngpm/data/logs/proxy-host-9\_access.log

![](https://img-cdn.limour.top/2022/08/05/62eceadc5bfb4.png)

sudo ip addr show docker0

```NGINX
if ($http_upgrade != "websocket") {
    return 404;
}
proxy_set_header Connection "upgrade";
proxy_read_timeout 300s;
proxy_redirect off;
```

## 第三步 CFT进行CDN

*   AWS服务搜索[CloudFront](https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-west-1#/)
*   创建服务，源写前面DNS解析的子域名，其他根据自己情况填写
*   部署后得到一个xxx.cloudfront.net的域名，将其添加的NPM面板的域名后面
*   修改address和SNI为xxx.cloudfront.net
*   后续address可以改为某个速度快的ip
*   这样就完成了博客的加速了