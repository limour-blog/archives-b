---
title: 封面图片API解决跨域问题
tags: []
id: '2130'
categories:
  - - uncategorized
date: 2023-02-06 10:50:57
---

## 新建CloudFlare转换规则

![](https://img-cdn.limour.top/i/2023/02/06/63e0dabf3772e.png)

## 修改响应头

![](https://img-cdn.limour.top/i/2023/02/06/63e0db016e756.png)

![](https://img-cdn.limour.top/i/2023/02/06/63e0db45a2176.png)

*   "Access-Control-Allow-Origin": "\*"

*   "Access-Control-Allow-Methods": "GET,HEAD,POST,OPTIONS"

*   "Access-Control-Max-Age": "86400"