---
title: Nginx Proxy Manager进阶：隐藏真实IP
tags: []
id: '2166'
categories:
  - - uncategorized
date: 2023-03-16 03:52:29
---

某AI的API在[反代](https://occdn.limour.top/2607.html)时，如果不小心泄露了原始的访问IP，很可能被封账号。这里在Nginx Proxy Manager里屏蔽掉一些可能泄露原始访问IP的header。

![](https://img-cdn.limour.top/i/2023/03/16/6412923b21b97.png)

```nginx
proxy_set_header X-Real-IP "";
proxy_set_header X-Forwarded-For "";
proxy_set_header CF-Connecting-IP "";
proxy_set_header Cookie "";
proxy_set_header CF-IPCOUNTRY "US";
proxy_hide_header CF-Connecting-IP;
proxy_hide_header X-Real-IP;
proxy_hide_header X-Forwarded-For;
proxy_hide_header Cookie;
proxy_hide_header CF-IPCOUNTRY;
```