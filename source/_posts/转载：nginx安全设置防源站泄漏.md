---
title: 转载：Nginx安全设置防源站泄漏
tags: []
id: '1213'
categories:
  - - 建站探索
date: 2021-11-01 23:40:54
---

[BT宝塔Nginx设置只允许域名访问禁止IP访问 SSL同步设置防源站泄漏](https://www.shopee6.com/web/web-tutorial/bt-nginx-disable-ip-access-ssl.html)

## 第一步 申请假证书

```
-----BEGIN CERTIFICATE-----
MIIDITCCAsagAwIBAgIUTcEWLzynkLCFCoAC1iDH2vG3EkYwCgYIKoZIzj0EAwIw
gY8xCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T
YW4gRnJhbmNpc2NvMRkwFwYDVQQKExBDbG91ZEZsYXJlLCBJbmMuMTgwNgYDVQQL
Ey9DbG91ZEZsYXJlIE9yaWdpbiBTU0wgRUNDIENlcnRpZmljYXRlIEF1dGhvcml0
eTAeFw0xOTAxMTMxNDMxMDBaFw0zNDAxMDkxNDMxMDBaMGIxGTAXBgNVBAoTEENs
b3VkRmxhcmUsIEluYy4xHTAbBgNVBAsTFENsb3VkRmxhcmUgT3JpZ2luIENBMSYw
JAYDVQQDEx1DbG91ZEZsYXJlIE9yaWdpbiBDZXJ0aWZpY2F0ZTBZMBMGByqGSM49
AgEGCCqGSM49AwEHA0IABAg/hZ9lDHj/f+0jDRAN23TkNEqIi46mCGnwZVD3glxL
l+a1mpfXLHSEFTipnSyQgmvkPYzQGaEIFD0q6W/ZgMujggEqMIIBJjAOBgNVHQ8B
Af8EBAMCBaAwHQYDVR0lBBYwFAYIKwYBBQUHAwIGCCsGAQUFBwMBMAwGA1UdEwEB
/wQCMAAwHQYDVR0OBBYEFCEZF6Eyem01XPbgwr6DXLZV1qsQMB8GA1UdIwQYMBaA
FIUwXTsqcNTt1ZJnB/3rObQaDjinMEQGCCsGAQUFBwEBBDgwNjA0BggrBgEFBQcw
AYYoaHR0cDovL29jc3AuY2xvdWRmbGFyZS5jb20vb3JpZ2luX2VjY19jYTAjBgNV
HREEHDAaggwqLmRuc3BvZC5jb22CCmRuc3BvZC5jb20wPAYDVR0fBDUwMzAxoC+g
LYYraHR0cDovL2NybC5jbG91ZGZsYXJlLmNvbS9vcmlnaW5fZWNjX2NhLmNybDAK
BggqhkjOPQQDAgNJADBGAiEAnrequCk/QZOOrcPH6C3Hgcy4SPNUy5rQtku/aYkj
qQoCIQCN6IyYNiXuwG+8jUgJrveiirBjiz2jXZSTEfVAyibjTg==
-----END CERTIFICATE-----

-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgK0HE3hTJQDg6p/fj
nS92eSuRKZEZ5F4grT6tWFKNYVmhRANCAAQIP4WfZQx4/3/tIw0QDdt05DRKiIuO
pghp8GVQ94JcS5fmtZqX1yx0hBU4qZ0skIJr5D2M0BmhCBQ9Kulv2YDL
-----END PRIVATE KEY-----
```

## 第二步 部署假页面并使用假证书

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/image.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/image.webp)

## 第三步 修改默认站点

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/image-1.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/image-1.webp)

## 第四步 修改返回代码

```
return 444;
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/image-2.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/image-2.webp)

## 第五步 安全检测

[绕过CDN寻找网站真实IP的方法汇总](https://zhuanlan.zhihu.com/p/33440472)