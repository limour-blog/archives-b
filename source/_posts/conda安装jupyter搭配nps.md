---
title: conda安装Jupyter搭配NPS
tags: []
id: '2047'
categories:
  - - uncategorized
date: 2022-10-04 05:28:33
---

大量单细胞数据，峰值内存使用量来到了0.5T的水平，自己的小机器是跑不动了，只能用学校的集群。但是学校集群对于交互式绘图来说是很难用的，只好自行安装Jupyter然后用NPS进行内网穿透了。

## 安装包

*   conda create -n jupyter -c anaconda jupyter
*   conda activate jupyter

## 启动脚本

```bash
#!/usr/bin/bash
~/bin/npc -server=nps.blog.com:8024 -vkey=*** -type=tcp > ~/log/npc.log 2>&1 &
source activate jupyter
jupyter lab \
--ip='0.0.0.0' \
--no-browser \
--ServerApp.token="****" \
--port=19878 \
--NotebookNotary.db_file=':memory:'
```

*   nano j.sh
*   chmod +x j.sh
*   nohup ./j.sh > ~/log/j.log 2>&1 &