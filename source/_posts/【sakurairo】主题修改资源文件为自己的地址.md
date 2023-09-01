---
title: 【Sakurairo】主题修改资源文件为自己的地址
tags: []
id: '2071'
categories:
  - - uncategorized
date: 2022-10-24 14:34:42
---

先按[NPM面板：反代Jsdelivr](https://occdn.limour.top/2013.html)搭建私有的Jsdelivr加速服务，然后按[如何使用jsDelivr+Github 实现免费CDN加速?](https://zhuanlan.zhihu.com/p/346643522)，[F](https://github.com/Fuukei/Sakurairo_Vision)[ork](https://github.com/Fuukei/Sakurairo_Vision)发布一个**release**版本，记下加速地址。

*   源地址：https://s.nmxc.ltd删掉我/sakurairo\_vision/@2.5/xxx
*   加速地址：https://jscdn.limour.top/gh/Limour-dev/Sakurairo\_Vision/xxx

宝塔面板内按[Jsdelivr失效后的折腾](https://occdn.limour.top/1853.html#Nginx%E7%9B%B8%E5%85%B3)的方式添加替换规则

*   sub\_filter "s.nmxc.ltd删掉我/sakurairo\_vision/@2.5/" "jscdn.limour.top/gh/Limour-dev/Sakurairo\_Vision/";