---
title: Linux无root权限配置连接时自启后台程序
tags: []
id: '1290'
categories:
  - - 有趣技能
date: 2021-12-06 22:19:47
---

## 第一步 安装好xray

*   [https://limour.top/1140.html](https://limour.top/1140.html) 里有相应步骤
*   [https://limour.top/631.html](https://limour.top/631.html) 里有配置环境变量的步骤

## 第二步 进行配置

```
PIDS=`ps -ef grep 'xray run -c /home/gene/zl_liu/etc/xui2.json' grep -v grep  awk '{print $2}'`
if [ "$PIDS" != "" ]; then
echo "xray is runing!"
else
xray run -c /home/gene/zl_liu/etc/xui2.json >/home/gene/zl_liu/log/xray.log 2>&1 &
fi
```