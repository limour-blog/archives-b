---
title: Apache为反向代理添加用户验证
tags:
  - Apache
  - Authentication
  - Reverse Proxy
id: '350'
categories:
  - - 建站探索
date: 2020-07-21 23:46:55
---

先参见这篇文章的基础知识:[https://limour.top/109.html](https://limour.top/109.html)

```
<Location /info/>
AllowOverride AuthConfig
AuthType Basic
AuthName "This page is private"
AuthBasicProvider file
AuthUserFile /media/app/apasswd
require valid-user
</Location>
ProxyPass /info/ http://127.0.0.1:30101/
ProxyPassReverse /info/ http://127.0.0.1:30101/
```