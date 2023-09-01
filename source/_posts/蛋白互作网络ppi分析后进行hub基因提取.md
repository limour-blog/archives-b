---
title: 蛋白互作网络PPI分析后进行hub基因提取
tags: []
id: '1992'
categories:
  - - uncategorized
date: 2022-08-20 00:43:20
---

## 导出基因

```R
x <- readRDS('../../fig.1/C_ref_A_fiig.1_A/x.rds')
x <- unique(c(x$RRA_up,x$RRA_down))
x <- as.data.frame(x)
write.table(x = x, file = 'input.txt', row.names = F, quote = F)
```

## 进行PPI分析

[string数据库](https://string-db.org/)在线分析，导出互作表格

## 提取hub基因

```R
dat <- read.table('string_interactions.tsv', header = T)
names(dat)[c(1,2)] <- c('Sourse', 'Target')
dat <- dat[c('Sourse', 'Target', 'combined_score')]
all <- unique(c(dat$Sourse,dat$Target))
Adj <- matrix(nrow = length(all),ncol = length(all), data = 0)
rownames(Adj) <- all
colnames(Adj) <- all
for(i in 1:nrow(dat)){
    s <- dat[i,'Sourse']
    t <- dat[i,'Target']
    w <- dat[i,'combined_score']
    Adj[s,t] <- w
    Adj[t,s] <- w
}
tmp <- rowSums(Adj)
tmp <- data.frame(tmp)
tmp$symbol <- rownames(tmp)
tmp <- tmp[order(tmp$tmp,decreasing = T),]
hub <- tmp[1:200,'symbol']
hub_Adj <- Adj[hub,hub]
```