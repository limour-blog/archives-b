---
title: R语言：数据集分割
tags: []
id: '1991'
categories:
  - - uncategorized
date: 2022-08-19 12:48:06
---

## 安装补充包

*   [conda activate ggsurvplot](https://occdn.limour.top/2215.html)
*   conda install -c conda-forge r-catools -y

## 划分训练集和验证集

```R
library("caTools")
dat <- readRDS('../data/dat.rds')
set.seed(123)
split = sample.split(substr(rownames(dat),1,2),SplitRatio = .8)
train_data = subset(dat, split == TRUE)
test_data  = subset(dat, split == FALSE)
```