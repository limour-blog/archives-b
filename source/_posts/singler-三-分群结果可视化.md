---
title: SingleR (三) 分群结果可视化
tags: []
id: '688'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-09-20 23:10:31
---

## 第一步 加载程辑包

```
# 0、加载程辑包
library(Seurat)
library(dplyr)
library(patchwork)
library(ggplot2)
```

## 第二步 配置路径

```
# 配置数据路径
root_path = "~/zlliu/R_data/21.07.15.IntegrateData"

# 配置结果保存路径
output_path = "~/zlliu/R_output/21.09.20.SingleR"
if (!file.exists(output_path)){dir.create(output_path)}

```

## 第三步 读入数据

```
# 1、读取数据和参考数据集
tp_sc = readRDS(file.path(root_path, 'hBLA_scRNAs.rds'))
```

## 第四步 加载SingleR结果

```
# 2、构造预测方法
f_get_pred <- function(scRNA, lc_i){
    
    # 如果已经计算过了，就不再重复计算，直接返回上一次的结果
    if(file.exists(sprintf( "%02d_hM1_subclass.txt", lc_i))){
        result_main_hpca <- read.table(sprintf( "%02d_hM1_subclass.txt", lc_i), stringsAsFactors = F)
        return (result_main_hpca)
    }
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

gc()
tp_sc_st <- 1
tp_sc_len <- length(tp_sc)

```

## 第五步 画UMAP图

```
# # 7、进行标准化并筛选可变基因
# for (lc_i in tp_sc_st:tp_sc_len) {
#   tp_sc[[lc_i]] <- NormalizeData(tp_sc[[lc_i]],
#                       normalization.method = "LogNormalize", scale.factor = 10000)
#   tp_sc[[lc_i]] <- FindVariableFeatures(tp_sc[[lc_i]],
#                       selection.method = "vst", nfeatures = 2000)
# }

# 7.3 Scaling the data
for (lc_i in tp_sc_st:tp_sc_len) {
    lc_all.genes <- rownames(tp_sc[[lc_i]])
    tp_sc[[lc_i]] <- ScaleData(tp_sc[[lc_i]], features = lc_all.genes)
}

# 11、进行PCA降维
for (lc_i in tp_sc_st:tp_sc_len) {
    tp_sc[[lc_i]] <- RunPCA(tp_sc[[lc_i]], features = VariableFeatures(object = tp_sc[[lc_i]]))
}

# 12、决定主维度
pca_dim <- list()
options(repr.plot.width=18, repr.plot.height=6)
lc_s = 0
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s + 1] <- 16
pca_dim[lc_s + 2] <- 16
lc_s = 2
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s+1] <- 19
pca_dim[lc_s+2] <- 17
lc_s = 4
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s+1] <- 18
pca_dim[lc_s+2] <- 16
lc_s = 6
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s+1] <- 17
pca_dim[lc_s+2] <- 19
lc_s = 7
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s+1] <- 19
pca_dim[lc_s+2] <- 16
lc_s = 9
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s+1] <- 16
pca_dim[lc_s+2] <- 19
lc_s = 11
ElbowPlot(tp_sc[[lc_s + 1]], ndims = 40) + ElbowPlot(tp_sc[[lc_s + 2]], ndims = 40)
pca_dim[lc_s+1] <- 16
pca_dim[lc_s+2] <- 16

# 13、UMAP降维可视化
for (lc_i in tp_sc_st:tp_sc_len) {
    tp_sc[[lc_i]] <- RunUMAP(tp_sc[[lc_i]], dims = 1:pca_dim[[lc_i]])
}

options(repr.plot.width=9, repr.plot.height=6)
options(ggrepel.max.overlaps = Inf)
lc_reduction = "umap"
lc_s = 0
DimPlot(tp_sc[[lc_s+1]], reduction = lc_reduction, group.by = 'hM1_subclass',  label = T, repel = T, label.size = 6) + labs(title = sprintf("%s of 10x", names(tp_sc)[[lc_s + 1]]))

# 14、查看所有分组，然后合并同类不同名的分组
tp_cName = c()
for (lc_i in tp_sc_st:tp_sc_len) {
    lc_cN = unique(unlist(tp_sc[[lc_i]]@meta.data['hM1_subclass']))
    tp_cName = c(tp_cName, lc_cN)
}
sort(unique(tp_cName))

f_listUpdateRe <- function(lc_obj, lc_bool, lc_item){
    lc_obj[lc_bool] <- rep(lc_item,times=sum(lc_bool))
    lc_obj
}

f_subSameName <- function(lc_scRNA, lc_group.by, lc_name_x, lc_name_y){
    for (lc_i in 1:length(lc_name_x)){
        lc_x = lc_name_x[lc_i]
        lc_y = lc_name_y[lc_i]
        lc_idx = (lc_scRNA@meta.data[lc_group.by] == lc_x)
        lc_scRNA@meta.data[lc_group.by] = f_listUpdateRe(lc_scRNA@meta.data[lc_group.by], lc_idx, lc_y)
    }
    lc_scRNA
}

for (lc_i in tp_sc_st:tp_sc_len) {
    tp_sc[[lc_i]] <- f_subSameName(tp_sc[[lc_i]], 'hM1_subclass',
              c('Astrocyte', 'Endothelial', 'LAMP5', 'Microglia', 'Oligodendrocyte', 'PVALB', 'SST', 'VIP'),
              c('Astro', 'Endo', 'Lamp5', 'Micro-PVM', 'Oligo', 'Pvalb', 'Sst', 'Vip'))
}

# 15.1、构造图片保存方法
f_image_output <- function(fileName, image, width=1920, height=1080){
  png(paste(fileName, ".png", sep=""), width = width, height = height)
  print(image)
  dev.off()
}

```

## 第六步 计算marker基因, 感觉效果不好

```
f_findMarkers <- function(lc_scRNA, lc_group.by){
    Idents(lc_scRNA) <- lc_scRNA@meta.data[,lc_group.by]
    lc_markers <- FindAllMarkers(lc_scRNA, only.pos = TRUE, group.by = lc_group.by, min.pct = 0.25, logfc.threshold = 0.25)
    lc_markers <- subset(lc_markers, subset = p_val_adj<0.1)
    lc_markers
}

```