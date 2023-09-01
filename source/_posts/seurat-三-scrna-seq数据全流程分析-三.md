---
title: Seurat (三) scRNA-seq数据全流程分析 (三)
tags: []
id: '824'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-10-01 15:38:22
---

## 第九步 提取Neuron细胞群

```
sc_Neuron <- subset(x = scRNA, integrated_snn_res.0.5 %in% c("2","6","8","11","13","15"))
saveRDS(sc_Neuron, "sc_Neuron.rds")
```

## 第十步 细化分析-手动

```
f_draw_VariableFeatures <- function(sObject){
    # sObject <- FindVariableFeatures(sObject, selection.method = "vst", nfeatures = 2000)
    # Identify the 10 most highly variable genes
    top10 <- head(VariableFeatures(sObject), 10)
 
    # plot variable features with and without labels
    plot1 <- VariableFeaturePlot(sObject)
    plot2 <- LabelPoints(plot = plot1, points = top10, repel = TRUE)
    plot2
}
 
options(repr.plot.width=8, repr.plot.height=6)
sc_Neuron <- FindVariableFeatures(sc_Neuron, selection.method = "vst", nfeatures = 2000)
f_draw_VariableFeatures(sc_Neuron)
 
sc_Neuron <- RunPCA(sc_Neuron, features = VariableFeatures(object = sc_Neuron))
options(repr.plot.width=12, repr.plot.height=6)
ElbowPlot(scRNA, ndims = 50)
pca_dim = 30
 
sc_Neuron <- RunUMAP(sc_Neuron, dims = 1:pca_dim)
lc_reduction = "umap"
options(repr.plot.width=9, repr.plot.height=9)
DimPlot(sc_Neuron, reduction = lc_reduction, group.by = 'orig.ident',  label = T, repel = T, label.size = 6) + labs(title = "UMAP reduction of brain regions")
 
options(repr.plot.width=9, repr.plot.height=18)
f_UMAP_more(sc_Neuron, c('hM1_hmca_class', 'manual_2'))
 
# 14、划分细胞群
sc_Neuron <- FindNeighbors(sc_Neuron, dims = 1:pca_dim)
sc_Neuron <- FindClusters(sc_Neuron, resolution = 1)
options(repr.plot.width=9, repr.plot.height=9)
DimPlot(sc_Neuron, reduction = lc_reduction)
 
sc_Neuron <- FindClusters(sc_Neuron, resolution = 2)
options(repr.plot.width=9, repr.plot.height=9)
DimPlot(sc_Neuron, reduction = lc_reduction)
 
# 16、寻找显著基因
Neuron_markers <- FindAllMarkers(sc_Neuron, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)
significant_Neuron_markers <- subset(Neuron_markers, subset = p_val_adj<0.05)
write.csv(significant_Neuron_markers, "significant_Neuron_markers.csv")
 
gl_ExN_InN <- rbind(gl_Markers_ExN, gl_Markers_InN)
 
tp_image <- f_get_m_p_a(sc_Neuron, significant_Neuron_markers, gl_ExN_InN)
f_image_output('gl_ExN_InN',tp_image, width = 2160, height = 2160)
 
Idents(sc_Neuron) <- sc_Neuron[['integrated_snn_res.2']]
f_setDistinction(sc_Neuron, "Manual_distinction_3.csv", significant_Neuron_markers, gl_ExN_InN)
 
Idents(sc_Neuron) <- sc_Neuron[['integrated_snn_res.2']]
sc_Neuron <- f_add_annotation(sc_Neuron, 'manual_3', "Manual_distinction_3.csv")
 
options(repr.plot.width=9, repr.plot.height=18)
f_UMAP_more(sc_Neuron, c('hM1_hmca_class', 'manual_3'))
```

## 第十一步 细化分析 -自动