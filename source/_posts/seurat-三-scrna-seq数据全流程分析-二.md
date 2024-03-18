---
title: Seurat (三) scRNA-seq数据全流程分析 (二)
tags: []
id: '800'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-10-01 03:54:52
---

## 第五步 手动标注 (一) marker表达可视化

```
# 4、构造图片保存方法
f_image_output <- function(fileName, image, width=1920, height=1080, lc_pdf=T, lc_resolution=72){
    if(lc_pdf){
        width = width / lc_resolution
        height = height / lc_resolution
        pdf(paste(fileName, ".pdf", sep=""), width = width, height = height)
    }else{
        png(paste(fileName, ".png", sep=""), width = width, height = height)
    }
    print(image)
    dev.off()
}

# 14、划分细胞群
scRNA <- FindNeighbors(scRNA, dims = 1:pca_dim)
scRNA <- FindClusters(scRNA, resolution = 0.5)
DimPlot(scRNA, reduction = lc_reduction)

# 16、寻找显著基因
all_markers <- FindAllMarkers(scRNA, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)
write.csv(significant_markers, "significant_markers.csv")

f_cluster_averages <- function(lc_scRNA){
    # 切分出Clusters
    lc_clusters <- SplitObject(lc_scRNA, split.by = 'ident')
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- lc_clusters[[lc_i]][[lc_clusters[[lc_i]]@active.assay]]@scale.data
    }
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- apply(lc_clusters[[lc_i]],1,mean)
    }
    lc_clusters <- data.frame(lc_clusters)
    scale(lc_clusters)
}
library(Hmisc)
library(corrplot)
f_corrplot <- function(lc_scdata){
    lc_cor <- rcorr(lc_scdata)
    lc_cor$P[is.na(lc_cor$P)]=0
    lc_cor$P <- -log10(lc_cor$P)
    lc_cor$P[is.infinite(lc_cor$P)] = max(lc_cor$P[!is.infinite(lc_cor$P)]) + 1
    corrplot(lc_cor$r, type="upper",tl.pos = "d",order = "AOE", p.mat = lc_cor$P, insig = "p-value")
    corrplot(lc_cor$r, add=TRUE, type="lower", method="number",order = "AOE", diag=FALSE,tl.pos="n", cl.pos="n")
}

# 3、准备好新的marker genes list
gl_MarkerGene <- read.csv(file.path(root_path, "cellType_Markers.csv"), header = F)
colnames(gl_MarkerGene) <- c('marker','celltype')

gl_Markers_ExN <- read.table(file.path(root_path, "ExN.csv"), header = F)
colnames(gl_Markers_ExN) <- c('marker','celltype')

gl_Markers_InN <- read.table(file.path(root_path, "InN.csv"), header = F)
colnames(gl_Markers_InN) <- c('marker','celltype')

gl_TS3 <- read.csv(file.path(root_path, "TableS3.csv"))
colnames(gl_TS3) <- c('marker','celltype')
gl_TS3$celltype <- substr(gl_TS3$celltype,1,3)

f_setRowName <- function(lc_df, lc_colName){
    lc_df <- lc_df[order(lc_df[[lc_colName]]),]
    tp_index <- duplicated(lc_df[[lc_colName]])
    lc_df <- lc_df[!tp_index,]
    rownames(lc_df) <- lc_df[[lc_colName]]
    lc_df
}

gl_MarkerGene <- f_setRowName(gl_MarkerGene, 'marker')
gl_TS3 <- f_setRowName(gl_TS3, 'marker')
gl_Markers_ExN <- f_setRowName(gl_Markers_ExN, 'marker')
gl_Markers_InN <- f_setRowName(gl_Markers_InN, 'marker')

# 18、绘制已知基因在不同cluster内的Violin plots
# 18.1 定义Violin plots绘制方法
f_get_cell_markers <- function(lc_cellType, lc_significant_markers, lc_MarkerGene){
    lc_features <- rownames(subset(lc_MarkerGene, subset = celltype == lc_cellType))
    lc_features <- lc_features[lc_features %in% lc_significant_markers$gene]
    lc_features
}
f_VlnPlot <- function(sObject, lc_cellType, lc_significant_markers, lc_MarkerGene) {
    lc_features <- f_get_cell_markers(lc_cellType, lc_significant_markers, lc_MarkerGene)
    if (length(lc_features)!=0){VlnPlot(sObject, features = lc_features)}
    else{ F } 
}

f_get_m_p <- function(sObject, lc_cellType, lc_significant_markers, lc_MarkerGene){
    lc_features <- f_get_cell_markers(lc_cellType, lc_significant_markers, lc_MarkerGene)
    if (length(lc_features)==0){return(NULL)}
    DotPlot(sObject, features = lc_features) + RotatedAxis() + 
    ggtitle(sprintf("%s markers in %s", lc_cellType, sObject@project.name)) + 
    theme(plot.title = element_text(hjust = 0.5))
}

# 18.2 绘制已知细胞类型的marker genes在本数据集中的pattern
f_get_all_celltype <- function(lc_MarkerGene){
    tp_cellTypes <- lc_MarkerGene$celltype # 获取所有细胞类型
    tp_cellTypes <- sort(tp_cellTypes) # 排序
    tp_cellTypes <- tp_cellTypes[!duplicated(tp_cellTypes)] #去重
    tp_cellTypes
}

f_get_m_p_a <- function(sObject, lc_significant_markers, lc_MarkerGene){
    lc_cellTypes <- f_get_all_celltype(lc_MarkerGene)
    tp_emm <- 2
    for (lc_j in 1:length(lc_cellTypes)){
        lc_image <- f_get_m_p(sObject, lc_cellTypes[lc_j], lc_significant_markers, lc_MarkerGene)
        if (!is.null(lc_image)){ break }
        tp_emm  <- tp_emm + 1
    }
    
    if (tp_emm > length(lc_cellTypes)){return(NULL)}
    
    for(lc_j in tp_emm:length(lc_cellTypes)){
        lc_image <- lc_image + f_get_m_p(sObject, lc_cellTypes[[lc_j]], lc_significant_markers, lc_MarkerGene)
    }
    lc_image <- lc_image + plot_layout(ncol = 3)
    lc_image
}

tp_image <- f_get_m_p_a(scRNA, significant_markers, gl_MarkerGene)
f_image_output('gl_MarkerGene',tp_image, width = 2160, height = 2160)
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/gl_MarkerGene.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/gl_MarkerGene.webp)

## 第六步 手动标注 (二) 生成默认标注

```
# 19.1 定义产生空标注的方法
f_unkown_marker_list <- function(sObject){
    tp_celltype  <- list()
    for (lc_i in 1:length(levels(sObject))){
        tp_celltype[[lc_i]] <- sprintf("Unkown_%02d", lc_i)
    }
    tp_celltype
}
f_marker_list_tp <- function(lc_significant_markers, lc_MarkerGene){
    tp_cluster_markers <- lc_MarkerGene[lc_significant_markers$gene,]
    tp_id <- which(rowSums(is.na(tp_cluster_markers)) == 0)
    lc_celltype <- tp_cluster_markers[,2]
    tp_cluster_markers <- cbind(lc_significant_markers, lc_celltype)
    tp_cluster_markers <- tp_cluster_markers[tp_id,]
    tp_cluster_markers
}
# 19.2 定义产生标注的方法
f_marker_list <- function(sObject, lc_significant_markers, lc_MarkerGene){
    lc_celltypes <- f_marker_list_tp(lc_significant_markers, lc_MarkerGene)[,c("cluster","lc_celltype")]
    lc_celltypes <- lc_celltypes[which(!duplicated(lc_celltypes[,1])),]
    #print(lc_celltypes)
    r_celltypes <- f_unkown_marker_list(sObject)
    for (lc_i in 1:nrow(lc_celltypes)){
        r_celltypes[[as.integer(lc_celltypes[lc_i,1])]] <- lc_celltypes[lc_i,2]
    }
    r_celltypes
}

f_setDistinction <- function(sObject, lc_filename, lc_significant_markers, lc_MarkerGene){
    if(!file.exists(lc_filename)){
        write.csv(data.frame(cluster_ID=levels(sObject),
                ident_1=unlist(f_marker_list(sObject, lc_significant_markers, lc_MarkerGene)),
                ident_2=unlist(f_unkown_marker_list(sObject))),
                lc_filename)
    }
}

Idents(scRNA) <- scRNA[['seurat_clusters']]
f_setDistinction(scRNA, "Manual_distinction_1.csv", significant_markers, gl_MarkerGene)

# 请参考已知基因在不同cluster内的Violin plots进行修正，修正在ident_2
# "Manual_distinction_1.csv"
f_marker_list_manual <- function(lc_filename){
    read.csv(lc_filename)[,"ident_2"]
}

f_add_annotation <- function(sObject, lc_metaName, lc_filename){
    lc_ids <- as.vector(unlist(f_marker_list_manual(lc_filename)))
    names(lc_ids) <- levels(sObject)
    sObject <- RenameIdents(sObject, lc_ids)
    sObject[[lc_metaName]] <- Idents(sObject)
    sObject
}

Idents(scRNA) <- scRNA[['seurat_clusters']]
scRNA <- f_add_annotation(scRNA, 'manual_1', "Manual_distinction_1.csv")

DimPlot(scRNA, reduction = lc_reduction, group.by = 'manual_1',  label = T, repel = T, label.size = 6) + labs(title = "UMAP reduction of clusters")
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/UMAP_manual.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/UMAP_manual.webp)

## 第七步 SingleR自动标注

```
saveRDS(scRNA, "scRNA.rds")
```

[SingleR (五) 不同参考集与混合参考集的对比](https://limour.top/692.html)

## 第八步 可视化对比

```
f_UMAP_more <- function(sObject, lc_group.by, lc_reduction="umap"){
    res <- (DimPlot(sObject, reduction = lc_reduction, group.by = lc_group.by[1], label = T, repel = T, label.size = 6) + 
    labs(title = lc_group.by[1]))
    for(lc_i in 2:length(lc_group.by)){
        res <- res/
        (DimPlot(sObject, reduction = lc_reduction, group.by = lc_group.by[lc_i], label = T, repel = T, label.size = 6) + 
        labs(title = lc_group.by[lc_i]))
    }
    res
}

f_UMAP_more(scRNA, c('hM1_hmca_class', 'hM1_class', 'hmca_class', 'manual_1'))

f_UMAP_more(sc_4, c('hM1_hmca_class', 'hM1_class', 'hmca_class', 'manual_1'))
```