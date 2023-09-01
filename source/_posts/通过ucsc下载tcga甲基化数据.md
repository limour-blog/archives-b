---
title: 通过UCSC下载TCGA甲基化数据
tags: []
id: '2003'
categories:
  - - uncategorized
date: 2022-09-03 14:42:00
---

[UCSC Xena](https://xenabrowser.net/datapages/)**是一个集分析、可视化、数据集下载等功能的在线数据分析和可视化平台**。 现有142个队列的1604个公共数据集包括TCGA, ICGC, TARGET, GTEx, CCLE等，不同的数据集之间精不同的标准化方法可以相互比较。（[陈浩](https://evvail.com/2021/12/25/2579.html)）

## 选择数据集

![](https://img-cdn.limour.top/2022/09/03/631366d26c67b.png)

![](https://img-cdn.limour.top/2022/09/03/63136707221ea.png)

## 下载数据

*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://tcga-xena-hub.s3.us-east-1.amazonaws.com/download/TCGA.PRAD.sampleMap%2FHumanMethylation450.gz

## 读取数据

```R
methy <- read.table( gzfile("TCGA.PRAD.sampleMap%2FHumanMethylation450.gz"), header = T, row.names = 1)
saveRDS(methy, 'methy_PRAD.rds')
```