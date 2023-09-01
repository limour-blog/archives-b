---
title: 构造Pseudo-bulk的表达量矩阵加速SingleR运行
tags: []
id: '2044'
categories:
  - - uncategorized
date: 2022-10-03 16:43:36
---

[The Human Cell Atlas](https://www.humancellatlas.org/) 和 [CZ CELLxGENE](https://cellxgene.cziscience.com/) 的单细胞数据集有些metadata里有细胞类型注释。我们使用[前面下载的数据集](https://occdn.limour.top/2387.html)来构建一个SingleR的参考集。

```R
sce <- readRDS('~/HumanCellAtlas/ProstateCellAtlas/cellxgene_Human_prostate.rds')
table(sce$tissue)
sce <- SeuratObject::UpdateSeuratObject(sce)
saveRDS(sce@meta.data, 'meta.rds')
sce@assays$RNA@counts
umi <- sce@assays$RNA@counts
sce <- Seurat::CreateSeuratObject(counts = umi, project = 'prostate', min.cells = 3, min.features = 200)
sce@meta.data <- readRDS('meta.rds')
all(colnames(sce) == rownames(sce@meta.data))
sce <- subset(sce, tissue == 'prostate gland')
gc()
table(sce$`Broad cell type`)
table(sce$`Granular cell type`)
table(sce$`Granular cell type`)
table(sce$`Tissue composition`)
table(sce$`Cell types level 2`)
table(sce$`Cell types level 3`)
sce$cell_t_1 <- droplevels(sce$`Tissue composition`)
all(as.character(sce$cell_t_1) == as.character(sce$`Tissue composition`))
sce$cell_t_2 <- droplevels(sce$`Cell types level 2`)
sce$cell_t_3 <- droplevels(sce$`Cell types level 3`)
sce <-  Seurat::NormalizeData(sce)
ref <- list()
ref$cell_t_1 <- Seurat::AverageExpression(sce,
                          group.by = "cell_t_1",
                          assays = "RNA")$RNA
ref$cell_t_2 <- Seurat::AverageExpression(sce,
                          group.by = "cell_t_2",
                          assays = "RNA")$RNA
ref$cell_t_3 <- Seurat::AverageExpression(sce,
                          group.by = "cell_t_3",
                          assays = "RNA")$RNA
saveRDS(ref, 'singleR_prostate.rds')
```