---
title: RRA结果绘图
tags: []
id: '1955'
categories:
  - - uncategorized
date: 2022-08-08 20:33:40
---

## 安装包

*   \# [conda activate rplot](https://occdn.limour.top/1561.html)
*   \# conda env remove -n ggVennDiagram
*   \# conda env remove -n rsf
*   conda create -n rsf -c conda-forge r-sf=1.0\_4
*   conda activate rsf
*   library(sf)
*   \# install.packages("sf", version = "1.0-4")
*   install.packages("ggVennDiagram")
*   conda install -c conda-forge r-ggsci -y
*   conda install -c conda-forge r-irkernel -y
*   Rscript -e "IRkernel::installspec(name='ggVennDiagram', displayname='r-ggVennDiagram')"
*   conda install -c conda-forge r-venndiagram -y

## 数据准备

```R
x <- list()
r <- list()
r$cell <- readRDS('../A_ref_A_fiig.1_A/DEG.rds')
r$tissue <- readRDS('../B_ref_A_fiig.1_A/DEG.rds')
names(r$tissue)[3] <- 'symbol'
r_up <- lapply(r, FUN = function(x){subset(x, log2FoldChange > 0)})
x$cell_up <- r_up$cell$symbol
x$RRA_up <- readRDS('r_up.rds')
x$RRA_up <- x$RRA_up$Name
x$tissue_up <- r_up$tissue$symbol
r_dn <- lapply(r, FUN = function(x){subset(x, log2FoldChange < 0)})
x$cell_down <- r_dn$cell$symbol
x$RRA_down <- readRDS('r_dn.rds')
x$RRA_down <- x$RRA_down$Name
x$tissue_down <- r_dn$tissue$symbol
summary(x)
```

## 绘图1

```R
#载入所需的R包；
library(ggplot2)
library(ggsci)
library(sf)
library(ggVennDiagram)
color4 <- alpha("#99CC00",0.5)
ggVennDiagram(x[1:6], label_alpha=0) +
  scale_fill_gradient(low='white',high =color4)
```

## 绘图2

```R
venn.plot <- venn.diagram(
    x = x[1:3],
    filename = NULL,
    cex = 2.5,
    cat.cex = 2.5,
    cat.dist = c(0.07, 0.07, 0.02),
    cat.pos = c(-20, 20, 20),
    alpha = 0.5,
    fill = c("#99CC00", "#c77cff", '#f8766d')
);
grid.draw(venn.plot)
venn.plot <- venn.diagram(
    x = x[4:6],
    filename = NULL,
    cex = 2.5,
    cat.cex = 2.5,
    cat.dist = c(0.07, 0.07, 0.02),
    cat.pos = c(-20, 20, 20),
    alpha = 0.5,
    fill = c("#99CC00", "#c77cff", '#f8766d')
);
grid.draw(venn.plot)
```