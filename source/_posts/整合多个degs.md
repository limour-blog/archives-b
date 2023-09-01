---
title: 整合多个DEGs
tags: []
id: '1031'
categories:
  - - 单细胞下游分析
  - - 生物信息学
  - - 组织测序分析
date: 2021-10-11 14:37:56
---

## 第一步 读入数据

```
# 配置数据路径
root_path = "~/zlliu/R_data/TCGA"
 
# 配置结果保存路径
output_path = root_path
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

su.d <- subset(read.csv('seurat.csv'), cluster=='CRPC')
su.f <- with(su.d, mean(abs(avg_log2FC)))
rownames(su.d) <- su.d$X
tcga <- readRDS('tcga.rds')
gse <- readRDS('gse.rds')

```

## 第二步 规范数据

```
degs = list()

degs$scRNA <- su.d[,c('avg_log2FC','p_val_adj','pct.1')]
colnames(degs$scRNA) <- c('logFC','q','m')

degs$tcga <- tcga$deg[,c('log2FoldChange','padj','baseMean')]
colnames(degs$tcga) <- c('logFC','q','m')
degs$tcga$logFC <- -degs$tcga$logFC # 转到PRAD为对照
degs$tcga$m <- log2(degs$tcga$m) # 取对数

degs$geo <- gse$deg[,c('logFC', 'adj.P.Val', 'AveExpr')]
colnames(degs$geo) <- c('logFC','q','m')

```

## 第三步 整合数据

```
degs.u <- list()
degs.d <- list()

degs.u$scRNA <- subset(degs$scRNA, logFC > 0)
degs.d$scRNA <- subset(degs$scRNA, logFC < 0)

degs.u$tcga <- subset(degs$tcga, logFC > 0)
degs.d$tcga <- subset(degs$tcga, logFC < 0)

degs.u$geo  <- subset(degs$geo , logFC > 0)
degs.d$geo  <- subset(degs$geo , logFC < 0)

degs.fm <- list()
degs.ffc <- list()

degs.fm$scRNA <- with(degs$scRNA, mean(m) - 2*sd(m))
degs.fm$tcga <- with(degs$tcga, mean(m)- 3*sd(m))
degs.fm$geo <- with(degs$geo, mean(m)- 4*sd(m))

degs.ffc$scRNA <- with(degs$scRNA, mean(abs(logFC)) + 0*sd(abs(logFC)))
degs.ffc$tcga <- with(degs$tcga, mean(abs(logFC)) + 0*sd(abs(logFC)))
degs.ffc$geo  <- with(degs$geo , mean(abs(logFC)) + 0*sd(abs(logFC)))
degs.ffc$lo  <- with(degs$lo , mean(abs(logFC)) + 1*sd(abs(logFC)))

degs.lo <- subset(degs$lo, abs(logFC) > degs.ffc$lo & q <0.05)
degs.u$scRNA <- subset(degs.u$scRNA , abs(logFC) > degs.ffc$scRNA & q <0.05)
degs.d$scRNA <- subset(degs.d$scRNA, abs(logFC) > degs.ffc$scRNA & q <0.05)
degs.u$tcga  <- subset(degs.u$tcga  , abs(logFC) > degs.ffc$tcga  & q <0.05)
degs.d$tcga  <- subset(degs.d$tcga , abs(logFC) > degs.ffc$tcga  & q <0.05)
degs.u$geo <- subset(degs.u$geo, abs(logFC) > degs.ffc$geo  & q <0.05)
degs.d$geo <- subset(degs.d$geo, abs(logFC) > degs.ffc$geo  & q <0.05)

```

## 第四步 可视化

```
library(RColorBrewer)
library(ggplot2)
# options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
# BiocManager::install('ggvenn')
library(ggvenn)
col_Paired <- colorRampPalette(brewer.pal(12, "Paired"))
f_venn <- function(lc_l, lc_c=NULL){
    if(length(lc_c)==0){lc_c = names(lc_l)}
    lc_col <- col_Paired(length(lc_l))
    ggvenn(lc_l,lc_c, show_percentage = T,
       stroke_color = "white",
       fill_color = lc_col,
       set_name_color = lc_col)
}
```

```
f_venn(u)
f_venn(d)
```