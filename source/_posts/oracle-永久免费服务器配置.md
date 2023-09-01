---
title: Oracle 永久免费服务器配置
tags: []
id: '1086'
categories:
  - - 建站探索
date: 2021-10-16 14:22:07
---

## 第一步 用自己生成的公钥创建实例

[https://zhuanlan.zhihu.com/p/352736372](https://zhuanlan.zhihu.com/p/352736372)

## 第二步 配置开机启动脚本

[https://zhangyaoyu.com/archives/399/](https://zhangyaoyu.com/archives/399/)

```
#!/bin/bash
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
iptables -F
```

## 第三步 安装宝塔面板

[https://pangwu86.com/posts/2992442120/](https://pangwu86.com/posts/2992442120/)