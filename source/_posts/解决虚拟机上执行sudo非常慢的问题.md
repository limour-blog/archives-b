---
title: 解决虚拟机上执行sudo非常慢的问题
tags: []
id: '1975'
categories:
  - - uncategorized
date: 2022-08-14 12:36:21
---

之前[用VMware安装了ubuntu](https://occdn.limour.top/2179.html)，一切都挺好，就是sudo运行起来等半天，跟[这里的描述类似](https://forum.ubuntu.org.cn/viewtopic.php?t=487728)，在大佬指路下找到了[解决方案](https://blog.csdn.net/longyinyushi/article/details/50527948)。

*   hostname 命令得到自己的主机名，如vmubuntu
*   sudo nano /etc/hosts 修改hosts文件
*   127.0.0.1 localhost vmubuntu 将vmubuntu添加到127.0.0.1后面
*   果然sudo不再卡顿了