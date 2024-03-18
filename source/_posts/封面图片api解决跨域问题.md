---
title: 封面图片API解决跨域问题
tags: []
id: '2130'
categories:
  - - uncategorized
date: 2023-02-06 10:50:57
---

## 新建CloudFlare转换规则

![](https://img.limour.top/archives_2023/2023/02/06/63e0dabf3772e.webp)

## 修改响应头

![](https://img.limour.top/archives_2023/2023/02/06/63e0db016e756.webp)

![](https://img.limour.top/archives_2023/2023/02/06/63e0db45a2176.webp)

*   "Access-Control-Allow-Origin": "\*"

*   "Access-Control-Allow-Methods": "GET,HEAD,POST,OPTIONS"

*   "Access-Control-Max-Age": "86400"