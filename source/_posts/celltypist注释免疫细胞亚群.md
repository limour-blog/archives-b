---
title: CellTypist注释免疫细胞亚群
tags: []
id: '1408'
categories:
  - - 单细胞下游分析
date: 2022-01-17 07:05:52
---

## 第一步 依赖安装

按 [原神 AI全自动钓鱼 使用方法](https://limour.top/1308.html) 配置conda镜像和pip镜像  
按 [R安装包前的一些配置](https://limour.top/1292.html) 配置R镜像

*   conda install scanpy
*   pip3 install celltypist -i https://pypi.tuna.tsinghua.edu.cn/simple
*   install.packages("reticulate")
*   celltypist.models.download\_models(force\_update = F)

## 第二步 加载程辑包和数据

[使用 CellTypist 进行免疫细胞类型分类](https://cloud.tencent.com/developer/article/1926662)

```r
library(reticulate)
scanpy = import("scanpy")
celltypist = import("celltypist")
pandas <- import("pandas")
numpy = import("numpy")
celltypist$models$download_models(force_update = F) 

library(Seurat)
sce <- readRDS("~/gene/upload/yy_zhang_data/scRNA-seq/pca.celltype.rds")
sce@meta.data
immune <- subset(sce,immune_annotation=="immune")
```

## 第三步 seurat转scanpy

[Seurat对象、SingleCellExperiment对象和scanpy对象的转化](https://www.jianshu.com/p/c438d545f696)

[Scanpy 使用原则](https://scanpy.readthedocs.io/en/stable/usage-principles.html)

```r
# 数据矩阵, scanpy与Seurat的行列定义相反
adata.X = numpy$array(t(as.matrix(immune[['RNA']]@counts)))
# 对每个细胞的观察
adata.obs = pandas$DataFrame(immune@meta.data[colnames(immune[['RNA']]@counts),])
# 对基因矩阵的注释
adata.var = pandas$DataFrame(data.frame(gene = rownames(immune[['RNA']]@counts), row.names = rownames(immune[['RNA']]@counts)))

# 组装AnnData对象
adata = scanpy$AnnData(X = adata.X, obs=adata.obs, var=adata.var)
```

## 第四步 进行预测

```r
model = celltypist$models$Model$load(model = 'Immune_All_Low.pkl')
model$cell_types
scanpy$pp$normalize_total(adata, target_sum=1e4)
scanpy$pp$log1p(adata)
predictions = celltypist$annotate(adata, model = 'Immune_All_Low.pkl', majority_voting = T)
```

## 第五步 预测添加到seurat对象

```r
immune = AddMetaData(immune, predictions$predicted_labels)
```

## 第六步 可视化查看

```r
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)
library(ggplot2)

DimPlot(immune, reduction = 'umap', group.by = 'majority_voting',  label = T, repel = T, label.size = 6) + labs(title = "UMAP reduction of majority_voting")

options(repr.plot.width=24, repr.plot.height=8)
DimPlot(immune, reduction = 'umap', group.by = 'predicted_labels', label.size = 6) + labs(title = "UMAP reduction of predicted_labels")
```