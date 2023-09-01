---
title: SingleR (二) 使用参考数据集预测分群
tags: []
id: '678'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-09-20 03:06:23
---

## 第一步 构建 21.09.20.SingleR.R

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
root_path = "~/zlliu/R_data/21.07.15.IntegrateData"

# 配置结果保存路径
output_path = "~/zlliu/R_output/21.09.20.SingleR"
if (!file.exists(output_path)){dir.create(output_path)}

# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

# 1、读取数据和参考数据集
scRNAs = readRDS(file.path(root_path, 'hBLA_scRNAs.rds'))
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

# 2、构造预测方法
f_get_pred <- function(scRNA, lc_i){
    
    # 如果已经计算过了，就不再重复计算，直接返回上一次的结果
    if(file.exists(sprintf( "%02d_hM1_subclass.txt", lc_i))){
        result_main_hpca <- read.table(sprintf( "%02d_hM1_subclass.txt", lc_i), stringsAsFactors = F)
        return (result_main_hpca)
    }
    
    # 导出原始counts矩阵
    test.count=as.data.frame(scRNA[["RNA"]]@counts)
    
    # 保留共同基因
    common_hpca <- intersect(rownames(test.count), c(rownames(hM1.se), rownames(hmca)))
    test.count_forhpca <- test.count[common_hpca,]
    tp_idx = na.omit(match(common_hpca, rownames(hM1.se)))
    lc_hM1.se <- hM1.se[rownames(hM1.se)[tp_idx], ]
    tp_idx = na.omit(match(common_hpca, rownames(hmca)))
    lc_hmca <- hmca[rownames(hmca)[tp_idx], ]

    gc()
    
    # 生成test数据集
    test.count_forhpca.se <- SummarizedExperiment(assays=list(counts=test.count_forhpca))
    test.count_forhpca.se <- logNormCounts(test.count_forhpca.se)
    
    # 进行分类预测 大概一小时
    pred.main.hpca <- SingleR(test = test.count_forhpca.se, ref = list(m1=lc_hM1.se, mca=lc_hmca), labels = list(hM1.se$meta$subclass_label, hmca$meta$subclass_label), BPPARAM=MulticoreParam(32)) # 32CPUs

    gc()
    
    #构造返回结果
    result_main_hpca <- as.data.frame(pred.main.hpca$labels)
    result_main_hpca$CB <- rownames(pred.main.hpca)
    colnames(result_main_hpca) <- c('hM1_subclass', 'CB')
    write.table(result_main_hpca, sprintf( "%02d_hM1_subclass.txt", lc_i)) #保存下来，方便以后调用
    result_main_hpca
}

# 3、进行预测
tp_sc <- scRNAs
# rm(x10s)
gc()
tp_sc_st <- 1
tp_sc_len <- length(tp_sc)
for (lc_i in tp_sc_st:tp_sc_len) {
    f_get_pred(tp_sc[[lc_i]], lc_i)
}

# 4、读取预测信息的函数
f_set_pred <- function(scRNA, result_main_hpca){
    scRNA@meta.data$CB <- rownames(scRNA@meta.data)
#     print(scRNA@meta.data$CB)
    scRNA@meta.data=merge(scRNA@meta.data,result_main_hpca,by="CB")
#     print(result_main_hpca$CB)
    rownames(scRNA@meta.data)=scRNA@meta.data$CB
    scRNA
}

for (lc_i in tp_sc_st:tp_sc_len) {
    tp_sc[[lc_i]] <- f_set_pred(tp_sc[[lc_i]], f_get_pred(tp_sc[[lc_i]], lc_i))
}

# 5、查找marker
f_findMarkers <- function(lc_scRNA, lc_group.by){
    Idents(lc_scRNA) <- lc_scRNA@meta.data[,lc_group.by]
    lc_markers <- FindAllMarkers(lc_scRNA, only.pos = TRUE, group.by = lc_group.by, min.pct = 0.25, logfc.threshold = 0.25)
    lc_markers <- subset(lc_markers, subset = p_val_adj<0.05)
    lc_markers
}

tp_markers  <- list()
for (lc_i in 1:length(tp_sc)) {
    tp_markers[[lc_i]] <- f_findMarkers(tp_sc[[lc_i]], 'hM1_subclass')
}

gl_MarkerGene <- read.csv(file.path("~/zlliu/R_data/hBLA", "cellType_Markers.csv"), header = F)

f_matchKownMarker <- function(lc_markers_all, lc_markers_kown){
    lc_CellType <- lc_markers_kown[match(rownames(lc_markers_all), lc_markers_kown[,1]),2]
    cbind(lc_markers_all, lc_CellType)
}

for (lc_i in 1:length(tp_sc)) {
    write.csv(f_matchKownMarker(tp_markers[[lc_i]], gl_MarkerGene), sprintf( "%02d_significant_markers_%s.csv", lc_i, names(tp_sc)[lc_i]))
}

gl_MarkerGene <- read.csv("~/zlliu/R_data/hBLA/TableS3.csv", header = T)

```

## 第二步 构建 21.09.20.SingleR.pbs 并提交

```
#!/bin/bash
#PBS -q fat
#PBS -V
#PBS -o /home/rqzhang/zlliu/PBS/SingleR/SingleR.out
#PBS -e /home/rqzhang/zlliu/PBS/SingleR/SingleR.err
#PBS -l nodes=1:ppn=32
#PBS -r y
cd /home/rqzhang
Rscript /home/rqzhang/zlliu/PBS/SingleR/21.07.15.10x.hM1.se.R
echo $HOME
```

*   提交job：`qsub /home/rqzhang/zlliu/PBS/SingleR/21.09.20.SingleR.pbs`
*   查看所有任务：`qstat`