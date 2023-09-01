---
title: Seurat (九) hclust聚类分析
tags: []
id: '1236'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-11-14 23:30:43
---

## 第一步 常规操作

```r
library(Matrix)
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)

library(ggplot2)
library(reshape2)


options(repr.plot.width=12, repr.plot.height=12)
options(ggrepel.max.overlaps = Inf)

f_br_cluster_f <- function(sObject, lc_groupN){
    lc_filter <- unlist(unique(sObject[[lc_groupN]]))
    lc_filter <- lc_filter[!is.na(lc_filter)]
    lc_filter
}

f_metadata_removeNA <- function(sObject, lc_groupN){
    sObject@meta.data <- sObject@meta.data[colnames(sObject),]
    sObject <- subset(x = sObject, !!sym(lc_groupN)%in%f_br_cluster_f(sObject, lc_groupN))
    sObject
}

f_image_output <- function(fileName, image, width=1920, height=1080, lc_pdf=T, lc_resolution=72){
    if(lc_pdf){
        width = width / lc_resolution
        height = height / lc_resolution
        pdf(paste(fileName, ".pdf", sep=""), width = width, height = height)
    }else{
        png(paste(fileName, ".png", sep=""), width = width, height = height)
    }
    print(image)
    dev.off()
}

# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"
 
# 配置结果保存路径
output_path = "~/zlliu/R_data/21.11.14.hclust"
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

scRNA_split = readRDS("~/zlliu/R_output/21.09.21.SingleR/scRNA.rds")
scRNA_split <- f_metadata_removeNA(scRNA_split, 'Region')
```

## 第二步 计算hclust

```r
f_ggplot2_ti <- function(p, title){
    (p + ggtitle(title) + theme(plot.title = element_text(hjust = 0.5)))
}

library(ggdendro)
f_ggdendrogram <- function(hc){
    ggdendrogram(hc$h, rotate = T, size = 3)+theme(axis.text = element_text(size=14,face = "bold"))
}

f_hc <- function(lc_counts){
    d <- dist(t(lc_counts))
    h <- hclust(d)
    list(d=d, h=h)
}

f_hc_cop <- function(hc){
    cop <- cophenetic(hc$h)
    cor(cop, hc$d)
}


f_cluster_averages <- function(lc_scRNA, lc_metaN='ident'){
    # 切分出Clusters
    lc_clusters <- SplitObject(lc_scRNA, split.by = lc_metaN)
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- lc_clusters[[lc_i]][[lc_clusters[[lc_i]]@active.assay]]@scale.data
    }
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- apply(lc_clusters[[lc_i]],1,mean)
    }
    lc_clusters <- data.frame(lc_clusters)
    scale(lc_clusters)
}
```

```r
test_s <- SplitObject(scRNA_split, split.by = 'orig.ident')
for (n in names(test_s)){
    test = f_cluster_averages(test_s[[n]], "Region")
    hc <- f_hc(test)
    print(paste(n, hc %>% f_hc_cop()))
    print(f_hc(test) %>% f_ggdendrogram() %>% f_ggplot2_ti(paste0('hclust by gene of ', n)))
}
```

## 第三步 检验结果

```r
library(Hmisc)
library(ggcorrplot)
f_corrplot <- function(lc_scdata){
    lc_cor <- rcorr(lc_scdata)
    lc_cor$P[is.na(lc_cor$P)]=0
    ggcorrplot(lc_cor$r, type="full",hc.order = T, lab = T, p.mat = lc_cor$P)
}
```

```r
test = f_cluster_averages(scRNA_split, "Region")
f_corrplot(test) %>% f_ggplot2_ti(paste0('cor by gene of ', 'all'))
```