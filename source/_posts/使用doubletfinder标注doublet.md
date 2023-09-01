---
title: 使用DoubletFinder标注Doublet
tags: []
id: '2036'
categories:
  - - uncategorized
date: 2022-10-01 05:44:24
---

## 确定最佳分群

读入之前[使用metacell进行分群聚类](https://occdn.limour.top/2366.html)中的数据

```R
f_getBestPcs <- function(stdev){
    # Determine percent of variation associated with each PC
    pct <- stdev / sum(stdev) * 100
    # Calculate cumulative percents for each PC
    cumu <- cumsum(pct)
    # Determine which PC exhibits cumulative percent greater than 90% and % variation associated with the PC as less than 5
    co1 <- which(cumu > 90 & pct < 5)[1]
    co1
    # Determine the difference between variation of PC and subsequent PC
    co2 <- sort(which((pct[1:length(pct) - 1] - pct[2:length(pct)]) > 0.1), decreasing = T)[1] + 1
    # Minimum of the two calculation
    pcs <- min(co1, co2)
    pcs
}
f_plotBestClusters <- function(sce){
    sce <- Seurat::FindClusters(
        object = sce,
        resolution = c(seq(.1,1.6,.1)) #起始粒度，结束粒度，间隔
    )
    options(repr.plot.width = 12, repr.plot.height = 16)
    require(clustree)
    clustree::clustree(sce@meta.data, prefix = "SCT_snn_res.")
}
```

```R
sce <- readRDS('SRX8890106.rds')
sce@meta.data <- readRDS('SRX8890106_meta.rds')
sce <- Seurat::RunPCA(sce, assay="SCT", verbose = FALSE)
pcs <- f_getBestPcs(sce [["pca"]]@stdev)
sce <- Seurat::FindNeighbors(sce, reduction = "pca", 
                             dims = 1:pcs, verbose = FALSE)
f_plotBestClusters(sce)
```

## 进行分群

![](https://img-cdn.limour.top/2022/10/01/6337c9ef721ee.png)

f\_plotBestClusters

```R
sce <- Seurat::FindClusters(
    object = sce,
    resolution = 1.3 #读图得到最佳分辨率
)
sce <- Seurat::RenameIdents(sce, 
                            '6'='6', 
                            '12'='6',
                            '16'='6',
                            '2'='6',
                            '5'='6'
                           )
sce <- Seurat::RenameIdents(sce, 
                            '0'='0', 
                            '1'='0',
                            '9'='0'
                           )
table(Seurat::Idents(sce))
```

## 标注Doublet

![](https://img-cdn.limour.top/2022/09/29/633471b0dea83.png)

读表获取先验的Doublet占比

```R
f_Doublet_get_pK <- function(sce, pcs){
    sweep.res <- DoubletFinder::paramSweep_v3(sce, PCs = 1:pcs, sct = T, num.cores=4)
    sweep.stats <- DoubletFinder::summarizeSweep(sweep.res, GT = FALSE)
    bcmvn <- DoubletFinder::find.pK(sweep.stats)
    pK_bcmvn <- as.numeric(as.character(bcmvn$pK[which.max(bcmvn$BCmetric)]))
    pK_bcmvn
}
f_DoubletFinder <- function(sce, pcs, pK_bcmvn, DoubletRate, seurat_clusters){
    homotypic.prop <- DoubletFinder::modelHomotypic(seurat_clusters)   # 最好提供celltype
    nExp_poi <- round(DoubletRate*length(seurat_clusters)) 
    nExp_poi.adj <- round(nExp_poi*(1-homotypic.prop))
    sce <- DoubletFinder::doubletFinder_v3(sce, PCs = 1:pcs, 
                                           pN = 0.25, pK = pK_bcmvn, 
                                           nExp = nExp_poi.adj, reuse.pANN = FALSE, 
                                           sct = T)
    sce
}
```

```R
pK_bcmvn <- f_Doublet_get_pK(sce, pcs)
sce$seurat_clusters <- Idents(sce)
# ~8000 cells ~6.1% DoubletRate 
sce <- f_DoubletFinder(sce, pcs, pK_bcmvn, 0.061, sce$seurat_clusters)
saveRDS(sce@meta.data, 'SRX8890106_meta.rds')
```

## 可视化

```R
sce <- Seurat::RunUMAP(sce, reduction = "pca", 
                       dims = 1:30, verbose = FALSE)
options(repr.plot.width = 12, repr.plot.height = 6)
DimPlot(sce, reduction = "umap", label = T, repel = T,
        group.by = c("DF.classifications_0.25_0.04_416", 'seurat_clusters'))
```

![](https://img-cdn.limour.top/2022/10/01/6337d3a982439.png)