---
title: 蛋白质组学TCPA数据集使用记录
tags: []
id: '2006'
categories:
  - - uncategorized
date: 2022-09-04 16:25:12
---

## 获取数据

*   进入[TCPA的下载页面](https://tcpaportal.org/tcpa/download.html)选择感兴趣的L4数据
*   unzip TCGA-PRAD-L4.zip

## 清洗数据

[f\_dedup\_IQR](https://occdn.limour.top/2157.html)

```R
tcpa <- read.csv('tmp/TCGA-PRAD-L4.csv')
type <- as.numeric(substr(tcpa$Sample_ID, 14, 15))
tcpa <- subset(tcpa, type < 10) # tp
rowNa <- substr(tcpa$Sample_ID,1, 12)
tcpa <- f_dedup_IQR(tcpa[-(1:4)],rowNa)
tcpa
```

后续可以用[limma包进行差异分析](https://occdn.limour.top/2171.html)