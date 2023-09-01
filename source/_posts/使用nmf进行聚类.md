---
title: 使用NMF进行聚类
tags: []
id: '1956'
categories:
  - - uncategorized
date: 2022-08-08 22:37:04
---

## 安装包

*   [conda activate wgcna](https://occdn.limour.top/2095.html)
*   conda install -c conda-forge r-nmf -y

## 确定秩

```R
library(NMF)
dat <- readRDS('PRAD.rds')
res <- nmf(dat,2:7,nrun=10, seed=123)
pdf(file = 'nmf_sp.pdf', width = 12, height = 12);plot(res);dev.off()
```

## 绘制共识网络并提取分组

```R
res.3 <- nmf(dat,3,nrun=10, seed=123)
pdf(file = 'nmf_consensusmap.pdf', width = 6, height = 6);consensusmap(res.3);dev.off()
group <- predict(res.3)
group <- as.data.frame(group)
saveRDS(group, 'group.rds')
```