---
title: Seurat绘制3D-umap
tags: []
id: '1989'
categories:
  - - uncategorized
date: 2022-08-19 06:16:42
---

之前[在Rstudio-server上安装了seurat](https://occdn.limour.top/2228.html)，现在可以试试绘制3D-umap了

## 安装补充包

*   export R\_LIBS\_SITE=""
*   .libPaths('/home/rstudio/miniconda3/envs/r\_4\_1\_3/lib/R/library')
*   install.packages("plotly")

## 正式开始

```R
library(plotly)
library(Seurat)
sample13 <- readRDS("~/upload/zl_liu/work/Prognosis/scRNA/sample13.rds")
sample13 <- RunUMAP(sample13, dims = 1:10, n.components = 3L)
plot.data <- FetchData(object = sample13, vars = c("UMAP_1", "UMAP_2", "UMAP_3", "seurat_clusters"))
plot.data$label <- paste(rownames(plot.data))
# Plot your data, in this example my Seurat object had 21 clusters (0-20)
plot_ly(data = plot.data, 
        x = ~UMAP_1, y = ~UMAP_2, z = ~UMAP_3, 
        color = ~seurat_clusters, 
        colors = c("lightseagreen",
                   "gray50",
                   "darkgreen",
                   "red4",
                   "red",
                   "turquoise4",
                   "black",
                   "yellow4",
                   "royalblue1",
                   "lightcyan3",
                   "peachpuff3",
                   "khaki3",
                   "gray20",
                   "orange2",
                   "royalblue4",
                   "yellow3",
                   "gray80",
                   "darkorchid1",
                   "lawngreen",
                   "plum2",
                   "darkmagenta")[1:7],
        type = "scatter3d", 
        mode = "markers", 
        marker = list(size = 5, width=2), # controls size of points
        text=~label, #This is that extra column we made earlier for which we will use for cell ID
        hoverinfo="text") #When you visualize your plotly object, hovering your mouse pointer over a point shows cell names
```