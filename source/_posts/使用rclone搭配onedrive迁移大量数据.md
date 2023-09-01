---
title: 使用Rclone搭配OneDrive迁移大量数据
tags: []
id: '2048'
categories:
  - - uncategorized
date: 2022-10-04 09:21:05
---

之前在自己的小机器上分析，现在需要在[学校集群](https://occdn.limour.top/2398.html)进行分析，因此需要在两个没有公网ip且不互联的服务器之间转移大量数据。因此计划使用Rclone，通过OneDrive进行中转。

## 打包需要转移的数据

```R
data <- list()
ref_sce <- readRDS('~/upload/zl_liu/data/pca.rds')
data$zyy_umi <- ref_sce@assays$RNA@counts
data$zyy_meta <- ref_sce@meta.data
ref_sce <- readRDS('~/work_st/Prognosis/idea_2/fig3.2/fig6/sce.rds')
data$ch_umi <- ref_sce@assays$originalexp@counts
data$ch_meta <- ref_sce@meta.data
# tp_dir <- list(
#     SRX6887739 = '~/work_st/sce/GSE137829/res/SRX6887739/outs/filtered_feature_bc_matrix',
#     SRX6887740 = '~/work_st/sce/GSE137829/res/SRX6887740/outs/filtered_feature_bc_matrix',
#     SRX6887741 = '~/work_st/sce/GSE137829/res/SRX6887741/outs/filtered_feature_bc_matrix',
#     SRX6887742 = '~/work_st/sce/GSE137829/res/SRX6887742/outs/filtered_feature_bc_matrix',
#     SRX8890105 = '~/work_st/sce/GSE137829/res/SRX8890105/outs/filtered_feature_bc_matrix',
#     SRX8890106 = '~/work_st/sce/GSE137829/res/SRX8890106/outs/filtered_feature_bc_matrix'
# )
# counts <- Seurat::Read10X(data.dir = unlist(tp_dir))
# sce <- Seurat::CreateSeuratObject(counts, project = 'GSE137829',
#                             min.cells = 3, min.features = 200)
# data$GSE137829_umi <- sce@assays$RNA@counts
# data$GSE137829_meta <- sce@meta.data
tp_dir <- list(
    P1 = '~/work/GSE137829/GSM4089151_P1_gene_cell_exprs_table.txt.gz',
    P2 = '~/work/GSE137829/GSM4089152_P2_gene_cell_exprs_table.txt.gz',
    P3 = '~/work/GSE137829/GSM4089153_P3_gene_cell_exprs_table.txt.gz',
    P4 = '~/work/GSE137829/GSM4089154_P4_gene_cell_exprs_table.txt.gz',
    P5 = '~/work/GSE137829/GSM4711414_P5_gene_cell_exprs_table.txt.gz',
    P6 = '~/work/GSE137829/GSM4711415_P6_gene_cell_exprs_table.txt.gz'
)
sce <- list()
for (i in names(tp_dir)){
    tmp <- read.table(gzfile(tp_dir[[i]]), header = T)
    umi <- Matrix::as.matrix(x = tmp[-c(1,2)])
    umi <- Matrix::Matrix(data = umi, sparse = T)
    rownames(umi) <- tmp$Symbol
    sce[[i]] <- Seurat::CreateSeuratObject(umi, project = i,
                                min.cells = 3, min.features = 200)
}
sce <- Reduce(merge, sce)
data$geo_umi <- sce@assays$RNA@counts
data$geo_meta <- sce@meta.data
saveRDS(data, '22.10.04.rds')
```

## Rclone挂载OneDrive

*   conda activate jupyter
*   conda install -c conda-forge rclone -y

在两台服务器上[挂载同一个OneDrive](https://occdn.limour.top/2083.html)，第二台可以[直接使用第一台的配置](https://occdn.limour.top/2088.html)

## Rclone上传下载数据

*   rclone copy --ignore-existing --progress --ignore-errors --transfers=1 ./22.10.04.rds onedrive:tmp
*   rclone ls onedrive:tmp
*   rclone copy --ignore-existing --progress --ignore-errors --transfers=1 onedrive:tmp/22.10.04.rds .