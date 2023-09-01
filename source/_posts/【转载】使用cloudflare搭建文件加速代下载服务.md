---
title: 【转载】使用CloudFlare搭建文件加速代下载服务
tags: []
id: '2129'
categories:
  - - uncategorized
date: 2023-02-06 09:55:27
---

还在使用龟速一般的 `git clone`? 某些文件国内无法访问？部分网站下载速度极慢？

试试最新的代下载文件服务吧，服务地址：[https://pd.zwc365.com](https://pd.zwc365.com/)

**理论上支持所有基于 http 协议的链接加速**，包括 websocket （本站节点支持，cloudflare 节点暂不支持 websocket）

支持下载文件、支持 `git bash` 终端直接 `git clone` 项目

使用方式就是在现有的链接前，额外增加本站的代下线路：

*   本站代下 加速线路：https://pd.zwc365.com/
*   cloudflare 加速线路：https://pd.zwc365.com/cfworker/

在 workers 页面，需要将以下链接的文本内容完整的复制，并粘贴到网页的输入框中：

[请点击此链接打开网页并完整的复制内容到输入框中](https://pd.zwc365.com/https://raw.githubusercontent.com/zwc456baby/file-proxy/master/index.js)

[阅读原文](https://zwc365.com/2020/09/24/file-proxy-download)