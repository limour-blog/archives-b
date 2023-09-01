---
title: SingleR (四) 搭建测试环境
tags: []
id: '703'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-09-21 01:00:14
---

```
library(Seurat) ##
library(SingleR)
library(ggplot2)
library(reshape2)
library(SeuratData)
pbmc <- pbmc3k
library(BiocParallel) # 并行计算加速
hpca.se=HumanPrimaryCellAtlasData() ##第一次载入会下载数据集，可能会慢一些，后面在用时就不用下载了
Blue.se=BlueprintEncodeData() 

# 配置结果保存路径
output_path = "~/zlliu/R_output/21.09.21.SingleR"
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

hM1.se  <- list()
hmca <- list()

# 2、进行预测
data_for_SingleR = pbmc[["RNA"]]@data

lc_hM1.se <- Blue.se
lc_hmca <- hpca.se

hM1.se$meta$subclass_label <- Blue.se$label.main
hmca$meta$subclass_label <- hpca.se$label.main

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