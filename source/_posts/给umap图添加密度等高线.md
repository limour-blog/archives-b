---
title: 给UMAP图添加密度等高线
tags: []
id: '1287'
categories:
  - - R绘图奇技淫巧
date: 2021-12-05 14:11:40
---

## 第一步 加载程辑包并读入绘制好UMAP图的Seurat对象

```r
library(Seurat)
library(SeuratData)
library(Matrix)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)
 
library(ggplot2)
library(reshape2)

options(repr.plot.width=12, repr.plot.height=12)
options(ggrepel.max.overlaps = Inf)

# pbmc <- pbmc3k
# pbmc[["percent.mt"]] <- PercentageFeatureSet(pbmc, pattern = "^MT-")
# pbmc <- subset(pbmc, subset = nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5)
# pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000)
# pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000)
# pbmc <- ScaleData(pbmc, features = rownames(pbmc))
# pbmc <- RunPCA(pbmc, features = VariableFeatures(object = pbmc))
# pbmc <- FindNeighbors(pbmc, dims = 1:10)
# pbmc <- FindClusters(pbmc, resolution = 0.5)
# pbmc <- RunUMAP(pbmc, dims = 1:10)
# saveRDS(pbmc, 'pbmc.rds')

pbmc <- readRDS('pbmc.rds')
```

## 第二步 定义绘图方法

```r
f_umap_density2d <- function(umapdata){
    ret =  ggplot() + geom_point(data = umapdata,aes(x=UMAP_1, y=UMAP_2, colour = ident))
    for (o_ident in levels(umapdata$ident)){
        tmpdata = subset(umapdata, ident == o_ident)
        ret = ret + stat_density2d(data = tmpdata, mapping = aes(x=UMAP_1, y=UMAP_2))
    }
    ret
}

f_umap_density2d_  <- function(sObject){
    p_umap <- DimPlot(sObject, reduction = "umap")
    f_umap_density2d(p_umap$data) + p_umap$theme
}
```

## 第三步 绘图示例

`f_umap_density2d_(pbmc)`

[![](https://img-cdn.limour.top/blog_wp/2021/12/16386846371.png)](https://img-cdn.limour.top/blog_wp/2021/12/16386846371.png)