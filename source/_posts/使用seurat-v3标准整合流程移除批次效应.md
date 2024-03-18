---
title: 使用Seurat v3标准整合流程移除批次效应
tags: []
id: '1973'
categories:
  - - uncategorized
date: 2022-08-13 08:22:46
---

之前试了[文献提供的去除批次效应的代码](https://occdn.limour.top/2227.html)，发现效果非常差，还是用标准流程走一下看看吧

## 读入数据

```R
library(scran);
library(tidyverse);
library(Seurat);
sce <- readRDS('scRNA.rds')
sce <- as.Seurat(sce)
sceL <- SplitObject(object = sce, split.by = 'Batch')
table(sce[['Batch']])
```

### 分开的10x数据文件夹

```R
root_path = "."
f_read10x <- function(dirN, ...){
    tp_samples <- list.files(file.path(root_path, dirN))
    tp_dir <- file.path(root_path, dirN, tp_samples)
    names(tp_dir) <- tp_samples
    scRNA <- list()
    for (lc_ba in names(tp_dir)){
        counts <- Read10X(data.dir = tp_dir[[lc_ba]])
        scRNA[[lc_ba]] <- CreateSeuratObject(counts, project = lc_ba, ...)
        scRNA[[lc_ba]][["percent.mt"]] <- PercentageFeatureSet(scRNA[[lc_ba]], pattern = "^MT-")
        scRNA[[lc_ba]][["percent.ERCC"]] <- PercentageFeatureSet(scRNA[[lc_ba]], pattern = "^ERCC-")
    }
    scRNA
}
```

## 标准整合流程

```R
# sceL <- lapply(X = sceL, FUN = function(x){NormalizeData(testA.seu, normalization.method = "LogNormalize", scale.factor = 10000)})
sceL <- lapply(X = sceL, FUN = function(x){FindVariableFeatures(x, selection.method = "vst", nfeatures = 2000)})
sceL.anchors <- FindIntegrationAnchors(object.list = sceL, dims = 1:30)
sceL.integrated <- IntegrateData(anchorset = sceL.anchors, dims = 1:30)
saveRDS(sceL.integrated, 'sceL.integrated.rds')
```

## 最终效果

```R
sce <- as.SingleCellExperiment(sceL.integrated)
library(scater);
options(repr.plot.width=7, repr.plot.height=6)
plotTSNE(sce, colour_by="Batch")
```

![](https://img.limour.top/archives_2023/2022/08/13/62f75f0279859.webp)

emm，效果差强人意