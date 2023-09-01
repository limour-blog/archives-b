---
title: 无Root权限安装Chrome
tags: []
id: '760'
categories:
  - - 有趣技能
date: 2021-09-23 22:50:08
---

*   [下载并上传对应的rpm包](https://www.google.com/chrome/?platform=linux)

```
# 解压RPM包 到当前目录
rpm2cpio google-chrome-stable_current_x86_64.rpm cpio -idvm

# 创建链接
ln -s /home/rqzhang/zlliu/dev/chrome/opt/google/chrome/chrome /home/rqzhang/zlliu/bin/
```