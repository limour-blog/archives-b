---
title: Seurat (十) 同群细胞在不同脑区的DEGs
tags: []
id: '1302'
categories:
  - - 单细胞下游分析
date: 2021-12-14 04:55:24
---

## 第一步 读入数据

```r
library(Matrix)
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)

library(ggplot2)
library(reshape2)

# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"
 
# 配置结果保存路径
output_path = "~/zlliu/R_output/21.11.03.boxplot"
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

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

options(repr.plot.width=12, repr.plot.height=12)
options(ggrepel.max.overlaps = Inf)
f_icg_boxp <- function(lc_exp, lc_icg, lc_g, title = NULL){
    lc_exp_L = melt(lc_exp[lc_icg, rownames(lc_g)])
    lc_exp_L <- cbind(lc_exp_L, rownames(lc_exp_L))
    colnames(lc_exp_L)=c('value','sample')
    if(is.data.frame(lc_g)){
        lc_exp_L$group = lc_g[[1]]
    }else{
        lc_exp_L$group = lc_g
    }
    p=ggplot(lc_exp_L,aes(x=group,y=value,fill=group))+geom_boxplot()
    p=p+stat_summary(fun="mean",geom="point",shape=23,size=3,fill="red")
    p=p+theme_set(theme_set(theme_bw(base_size=20)))
    p=p+theme(text=element_text(face='bold'),axis.text.x=element_text(angle=30,hjust=1),axis.title=element_blank())
    if (length(title) == 0){title = lc_icg}
    p=p+ggtitle(title)+theme(plot.title = element_text(hjust = 0.5))
    p
}

f_bp_gcg <- function(sObject, lc_groupN, lc_labelN, gG){
    ss <- SplitObject(sObject, split.by = lc_groupN)
    lc_N =  names(ss)[1]
    p = f_icg_boxp(ss[[lc_N]][['RNA']]@data, gG, ss[[lc_N]][[lc_labelN]], sprintf('%s in %s', gG, lc_N))
    for (lc_N in names(ss)[-1]){
        p = p + f_icg_boxp(ss[[lc_N]][['RNA']]@data, gG, ss[[lc_N]][[lc_labelN]], sprintf('%s in %s', gG, lc_N))
    }
    p + plot_layout(ncol = 3)
}

f_br_cluster_f <- function(sObject, lc_groupN){
    lc_filter <- unlist(unique(sObject[[lc_groupN]]))
    lc_filter <- lc_filter[!is.na(lc_filter)]
    lc_filter
}

f_metadata_removeNA <- function(sObject, lc_groupN){
    sObject@meta.data <- sObject@meta.data[colnames(sObject),]
    sObject <- subset(x = sObject, !!sym(lc_groupN)%in%f_br_cluster_f(sObject, lc_groupN))
    sObject
}

scRNA_split = readRDS("~/zlliu/R_output/21.09.21.SingleR/scRNA.rds")
scRNA_split <- f_metadata_removeNA(scRNA_split, 'Region')
```

## 第二步 构建输出差异基因的函数

```r
f_ggplot2_ti <- function(p, title){
    (p + ggtitle(title) + theme(plot.title = element_text(hjust = 0.5)))
}

f_sc_DoHeatmap <- function(sObject, significant_markers, lc_groupN = 'ident'){
    if(nrow(significant_markers)<1){
        print(significant_markers)
        return()
    }
    significant_markers %>%
        group_by(cluster) %>%
        top_n(n = 10, wt = avg_log2FC) -> topn
    if(nrow(topn)<1){
        print(topn)
        return()
    }else{
        pN <- paste0(sObject@project.name,'_',lc_groupN,"_significant_markers")
        img = DoHeatmap(sObject, group.by=lc_groupN, features=topn$gene, disp.min=-1, disp.max=1) %>% f_ggplot2_ti(sObject@project.name)
        f_image_output(pN, img, lc_pdf = F)
    }
}

f_sc_degs <- function(sObject, lc_groupN){
    pN <- paste0(sObject@project.name,'_',lc_groupN,"_sObject_markers.csv")
    if(file.exists(pN)){
        print(pN)
        return()
    }
    Idents(sObject) <- sObject[[lc_groupN]]
    sObject_markers <- FindAllMarkers(sObject, min.pct = 0.25, logfc.threshold = 0.25)
    if(nrow(sObject_markers)<1  !('p_val_adj' %in% colnames(sObject_markers))){
        print(sObject_markers)
        return()
    }
    significant_markers <- subset(sObject_markers, subset = p_val_adj<0.05)
    write.csv(sObject_markers, paste0(sObject@project.name,'_',lc_groupN,"_sObject_markers.csv"))
    write.csv(significant_markers, paste0(sObject@project.name,'_',lc_groupN,"_significant_markers.csv"))
    f_sc_DoHeatmap(sObject,significant_markers,lc_groupN)

}
```

## 第三步 按细胞群切分并输出差异基因

```r
f_sc_DoHeatmap_sp <- function(sObject, lc_labelN, lc_groupN){
    ss <- SplitObject(sObject, split.by = lc_labelN)
    for (lc_N in names(ss)){
        sso <- ss[[lc_N]]
        sso@project.name  <- gsub('/', '.', lc_N)
        f_sc_degs(sso, lc_groupN)
    }
}
f_sc_DoHeatmap_sp(scRNA_split, 'hM1_hmca_class', 'Region')
```