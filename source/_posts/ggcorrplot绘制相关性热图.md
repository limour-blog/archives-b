---
title: ggcorrplot绘制相关性热图
tags: []
id: '1957'
categories:
  - - uncategorized
date: 2022-08-09 18:09:49
---

## 安装包

*   [conda activate wgcna](https://occdn.limour.top/2095.html)
*   conda install -c conda-forge r-ggcorrplot -y
*   conda install -c conda-forge r-ggsci -y
*   \# conda install -c conda-forge r-gridextra -y

## 颜色参考

```R
library("reshape2")

set.seed(42)
k <- 9
x <- diag(k)
x[upper.tri(x)] <- runif(sum(1:(k - 1)), 0, 1)
x_melt <- melt(x)

```

## 相关性热图

```R
library(Hmisc)
library(ggcorrplot)
library(ggsci)
library(gridExtra)
f_corrplot <- function(lc_scdata, scale=T, lab=F){
    lc_cor <- rcorr(lc_scdata)
    lc_cor$P[is.na(lc_cor$P)]=0
    if(scale){lc_cor$r[lc_cor$r<0] <- lc_cor$r[lc_cor$r<0] - (1 + min(lc_cor$r))}
    ggcorrplot(lc_cor$r, type="full",hc.order = T, lab = lab, p.mat = lc_cor$P)
}
options(repr.plot.width=8, repr.plot.height=8)
p <- f_corrplot(as.matrix(datExpr)) + scale_fill_material("cyan")
p
```