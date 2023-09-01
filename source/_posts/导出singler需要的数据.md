---
title: 导出SingleR需要的数据
tags: []
id: '2045'
categories:
  - - uncategorized
date: 2022-10-03 17:06:30
---

```R
tp_samples <- list.files('~/GEO/GSE193337')
tp_dir <- file.path('~/GEO/GSE193337', tp_samples)
names(tp_dir) <- tp_samples
counts <- Seurat::Read10X(data.dir = tp_dir)
sce <- Seurat::CreateSeuratObject(counts, project = 'prostate',
                            min.cells = 3, min.features = 200)
rm(counts)
gc()
table(sce$orig.ident)
sce <- Seurat::NormalizeData(sce)
logumi <- Seurat::GetAssayData(sce, slot="data")
saveRDS(logumi, 'logumi.rds')
```