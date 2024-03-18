---
title: FeedMe搭配TTRSS的一个坑
tags: []
id: '2172'
categories:
  - - uncategorized
date: 2023-03-22 03:29:34
---

海信墨水屏手机配合FeedMe读RSS订阅非常享受，但是直接用FeedMe的TTRSS登录功能无法正确同步，只能通过TTRSS的Fever API中转。同时api端点的域名也不是页面上显示的 `https://xxx/plugins.local/fever/` 而是 `https://xxx/plugins` 没有 `.local`。最终效果如图二，FeedMe设置里开启墨水屏优化，查看里设置布局为卡片。

![](https://img.limour.top/archives_2023/2023/03/22/641a7503cd3f2.webp)

![](https://img.limour.top/archives_2023/2023/03/22/641a745bb2eea.jpg)