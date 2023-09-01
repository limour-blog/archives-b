---
title: Seurat (六) 整合数据（二）
tags: []
id: '1014'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-10-09 16:58:11
---

## 第一步 常规配置

```
library(Matrix)
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)

rm(list = ls())
# 配置数据路径
root_path = "~/zlliu/R_data/TCGA"
 
# 配置结果保存路径
output_path = root_path
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

```

```
scRNA <- f_read10x('CRPC-PRAD')
```

## 第二步 质量控制

```
options(repr.plot.width=12, repr.plot.height=6)
options(ggrepel.max.overlaps = Inf)
library(ggplot2)
for (scRNAo in scRNA){
    print(VlnPlot(scRNAo, features = c("nFeature_RNA", "nCount_RNA", "percent.mt"), ncol = 3) + labs(title = scRNAo@project.name))
}

```

## 第三步 整合数据

```
f_go4pca <- function(scRNA){
    for (lc_ba in names(scRNA)){
        scRNA[[lc_ba]] <- scRNA[[lc_ba]] %>% NormalizeData() %>% FindVariableFeatures()
        all.genes <- rownames(scRNA[[lc_ba]])
        scRNA[[lc_ba]] <- ScaleData(scRNA[[lc_ba]], features = all.genes)
        scRNA[[lc_ba]] <- RunPCA(scRNA[[lc_ba]], features = VariableFeatures(object = scRNA[[lc_ba]]))
    }
    scRNA
}
f_go4group <- function(scRNA, mdN, mdNo='orig.ident', s=1, e=4){
    for (lc_ba in names(scRNA)){
        scRNA[[lc_ba]][[mdN]] <- substr(unlist(scRNA[[lc_ba]][[mdNo]]),s,e)
    }
    scRNA
}
```

```
scRNA <- f_go4group(scRNA, 'group')
scRNA <- f_go4group(scRNA, 'batch', s=5, e=5)
scRNA <- Reduce(function(...) merge(...), scRNA)
```

```
scRNA <- scRNA %>% NormalizeData() %>% FindVariableFeatures()
all.genes <- rownames(scRNA)
scRNA <- ScaleData(scRNA, features = all.genes)
scRNA <- RunPCA(scRNA, features = VariableFeatures(object = scRNA))
```

```
library(harmony)
scRNA = scRNA %>% RunHarmony("batch", plot_convergence = TRUE)
```

## 第四步 基于 Harmony 进行分群

```
scRNA <- scRNA %>% RunUMAP(reduction = "harmony", dims = 1:30)
scRNA <- scRNA %>% FindNeighbors(reduction = "harmony") %>%  FindClusters(resolution = 0.1)
scRNA[['t_RNA_snn_res.0.1']] <- Idents(scRNA)
Idents(scRNA) <- scRNA[['t_RNA_snn_res.0.1']]
all_markers <- FindAllMarkers(scRNA, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)
```