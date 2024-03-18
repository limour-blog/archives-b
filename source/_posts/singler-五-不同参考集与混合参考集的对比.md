---
title: SingleR (五) 不同参考集与混合参考集的对比
tags: []
id: '692'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-09-21 04:35:46
---

## 第一步 构建 21.09.21.SingleR.R

```
# 0、加载程辑包
library(Seurat)
library(dplyr)
library(patchwork)
 
# 加载 SingleR
library(SingleR)
library(SummarizedExperiment)
library(scater)
library(BiocParallel) # 并行计算加速
 
# 配置数据路径
root_path = "/home/rqzhang/rqzhang"
 
# 配置结果保存路径
output_path = "~/zlliu/R_output/21.09.21.SingleR"
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

# 1、读取数据和参考数据集
scRNA = readRDS("/home/rqzhang/rqzhang/SC_gene_integrate.rds")

if (!("hM1.se" %in% ls())){
    hM1.se <- readRDS("/home/rqzhang/zlliu/R_data/human_M1_10x/hM1.se.rds")
}
if (!("hmca" %in% ls())){
    hmca <- readRDS("/home/rqzhang/zlliu/R_data/Human_Multiple_Cortical_Areas_SMART-seq/hmca.rds")
}

# 1.1、裁剪数据集
f_noNull <- function(lc_se, lc_col){
    if(any(lc_se$meta["sample_name"] != colnames(lc_se))){
        return
    }
    lc_idx = which(lc_se$meta[lc_col] == "")
    lc_res = lc_se[, -lc_idx]
    lc_res$meta = lc_se$meta[-lc_idx,]
    lc_res
}
 
hmca <- f_noNull(hmca, "subclass_label")

# 1.2、修订标签
f_listUpdateRe <- function(lc_obj, lc_bool, lc_item){
    lc_obj[lc_bool] <- rep(lc_item,times=sum(lc_bool))
    lc_obj
}

f_subSameName <- function(lc_obj, lc_name_x, lc_name_y){
    for (lc_i in 1:length(lc_name_x)){
        lc_x = lc_name_x[lc_i]
        lc_y = lc_name_y[lc_i]
        lc_idx = (lc_obj == lc_x)
        lc_obj  = f_listUpdateRe(lc_obj , lc_idx, lc_y)
    }
    lc_obj
}

hM1.se$meta$subclass_label <- f_subSameName(hM1.se$meta$subclass_label,
          c('Astrocyte', 'Endothelial', 'LAMP5', 'Microglia', 'Oligodendrocyte', 'PVALB', 'SST', 'VIP'),
          c('Astro', 'Endo', 'Lamp5', 'Micro-PVM', 'Oligo', 'Pvalb', 'Sst', 'Vip'))
hmca$meta$subclass_label <- f_subSameName(hmca$meta$subclass_label,
          c('Astrocyte', 'Endothelial', 'LAMP5', 'Microglia', 'Oligodendrocyte', 'PVALB', 'SST', 'VIP'),
          c('Astro', 'Endo', 'Lamp5', 'Micro-PVM', 'Oligo', 'Pvalb', 'Sst', 'Vip'))


# 2、进行预测
data_for_SingleR = scRNA[["integrated"]]@data


# 保留共同基因
common_gene <- intersect(rownames(data_for_SingleR), c(rownames(hM1.se), rownames(hmca)))
data_for_SingleR <- data_for_SingleR[common_gene,]

tp_idx = na.omit(match(common_gene, rownames(hM1.se)))
lc_hM1.se <- hM1.se[rownames(hM1.se)[tp_idx], ]
tp_idx = na.omit(match(common_gene, rownames(hmca)))
lc_hmca <- hmca[rownames(hmca)[tp_idx], ]

# 进行分类预测
pred_1 <- SingleR(test = data_for_SingleR, ref = list(m1=lc_hM1.se, mca=lc_hmca),
                          labels = list(hM1.se$meta$subclass_label, hmca$meta$subclass_label),
                          BPPARAM=MulticoreParam(32)) # 32CPUs

pred_2 <- SingleR(test = data_for_SingleR, ref = lc_hM1.se,
                          labels = hM1.se$meta$subclass_label,
                          BPPARAM=MulticoreParam(32)) # 32CPUs

pred_3 <- SingleR(test = data_for_SingleR, ref = lc_hmca,
                          labels = hmca$meta$subclass_label,
                          BPPARAM=MulticoreParam(32)) # 32CPUs

# 保存结果
f_merge  <- function(lc_mergedList){
    Reduce(function(...) merge(..., by="CB"), lc_mergedList)
}

f_pred2meta <- function(lc_pred, lc_colname){
    lc_result  <- as.data.frame(lc_pred$labels)
    lc_result$CB  <- rownames(lc_pred)
    colnames(lc_result) <- c(lc_colname, 'CB')
    lc_result
}

pred_1  <- f_pred2meta(pred_1, "hM1_hmca_class")
pred_2  <- f_pred2meta(pred_2, "hM1_class")
pred_3  <- f_pred2meta(pred_3, "hmca_class")

```

## 第二步 构建PBS文件并提交

## 第三步 可视化

```
# 0、加载程辑包
library(Seurat)
library(dplyr)
library(patchwork)
library(ggplot2)

# 配置数据路径
root_path = "~/zlliu/R_output/21.09.21.SingleR"

# 配置结果保存路径
output_path = "~/zlliu/R_output/21.09.21.SingleR"
if (!file.exists(output_path)){dir.create(output_path)}

# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

# 1、读取数据
scRNA = readRDS("/home/rqzhang/rqzhang/SC_gene_integrate.rds")

# 2、读取预测信息
result <- read.table("hM1_hmca_class.txt", stringsAsFactors = F)

# 3、合并
scRNA@meta.data$CB <- rownames(scRNA@meta.data)
scRNA@meta.data=merge(scRNA@meta.data, result, by="CB")
rownames(scRNA@meta.data) <- scRNA@meta.data$CB

# 11、进行PCA降维
scRNA <- RunPCA(scRNA, features = VariableFeatures(object = scRNA))

# 12、决定主维度
options(repr.plot.width=18, repr.plot.height=6)
ElbowPlot(scRNA, ndims = 40)

pca_dim <- 24

# 13、UMAP降维可视化
scRNA <- RunUMAP(scRNA, dims = 1:pca_dim)

```

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/16321701111.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/16321701111.webp)