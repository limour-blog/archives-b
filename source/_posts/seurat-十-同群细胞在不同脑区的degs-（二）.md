---
title: Seurat (十) 同群细胞在不同脑区的DEGs （二）
tags: []
id: '1334'
categories:
  - - 单细胞下游分析
date: 2021-12-16 15:58:50
---

## 第一步 安装export包

*   [R安装包前的一些配置](https://limour.top/1292.html)
*   [Seurat (一) 安装](https://limour.top/643.html)
*   从 [https://github.com/tomwenseleers/export](https://github.com/tomwenseleers/export) 下载代码压缩包

```r
install.packages(c("officer", "rvg", "openxlsx", "flextable", "xtable", 
                   "rgl", "stargazer", "tikzDevice", "xml2", "broom"))

devtools::install_local("/home/rqzhang/zlliu/temp/export-master.zip")
```

## 第二步 读取数据

```r
library(Matrix)
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)
 
library(ggplot2)
library(reshape2)

library(openxlsx)
library(export)

# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"

# 配置结果保存路径
output_path = "~/zlliu/R_output/21.12.16.R2PPT_XLSX"
if (!file.exists(output_path)){dir.create(output_path)}

# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

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

## 第三步 批量输出图片和表格到一个文件

*   [R语言实现多种图像格式导出再编辑](https://cloud.tencent.com/developer/article/1476979)
*   [R语言中写入Excel的不同sheet表格](https://blog.csdn.net/yijiaobani/article/details/106154284)

```r
f_xlsx_output <- function(fileName, lc_xlsx){
    write.xlsx(lc_xlsx, fileName, rowNames=T)
}
f_ppt_output <- function(fileName, image){
    for (p in image){
        graph2ppt(x=p, file=fileName, width=9, aspectr=1.5, append = TRUE, vector.graphic=F)
    }
}
```

## 第四步 构造输出差异基因的函数

```r
f_mkdir_s <- function(lc_path){
    if (!file.exists(lc_path)){dir.create(lc_path)}
    setwd(lc_path)
    print(getwd())
}

f_mkdir_e <- function(){
    setwd(output_path)
    print(getwd())
}

f_ggplot2_ti <- function(p, title){
    (p + ggtitle(title) + theme(plot.title = element_text(hjust = 0.5)))
}
 
f_sc_DoHeatmap <- function(sObject, significant_markers, lc_groupN = 'ident', lc_listP){
    if(nrow(significant_markers)<1){
        print(significant_markers)
        return(lc_listP)
    }
    significant_markers %>%
        group_by(cluster) %>%
        top_n(n = 10, wt = avg_log2FC) -> topn
    if(nrow(topn)<1){
        print(topn)
        return(lc_listP)
    }else{
        pN <- paste0(sObject@project.name,'_',lc_groupN,"_significant_markers")
        img = DoHeatmap(sObject, group.by=lc_groupN, features=topn$gene, disp.min=-1, disp.max=1) %>% f_ggplot2_ti(sObject@project.name)
        lc_listP[[pN]] <- img
    }
    return(lc_listP)
}
 
f_sc_degs <- function(sObject, lc_groupN, lc_listR){
    Idents(sObject) <- sObject[[lc_groupN]]
    sObject_markers <- FindAllMarkers(sObject, min.pct = 0.25, logfc.threshold = 0.25)
    if(nrow(sObject_markers)<1  !('p_val_adj' %in% colnames(sObject_markers))){
        print(sObject_markers)
        return(lc_listR)
    }
    significant_markers <- subset(sObject_markers, subset = p_val_adj<0.05)
    SN <- paste0(sObject@project.name,'_',lc_groupN,"_sObject_markers")
    SN <- substr(SN, 1, 31)
    lc_listR$M[[SN]] <- sObject_markers
    SN <- paste0(sObject@project.name,'_',lc_groupN,"_significant_markers")
    SN <- substr(SN, 1, 31)
    lc_listR$SM[[SN]] <- significant_markers
    lc_listR$P <- f_sc_DoHeatmap(sObject,significant_markers,lc_groupN,lc_listR$P)
    return(lc_listR)
}

f_sc_DoHeatmap_sp <- function(sObject, lc_labelN, lc_groupN, sampleN){
    ss <- SplitObject(sObject, split.by = lc_labelN)
    lc_listR <- list()
    lc_listR$M <- list()
    lc_listR$SM <- list()
    lc_listR$P <- list()
    for (lc_N in names(ss)){
        sso <- ss[[lc_N]]
        sso@project.name  <- gsub('/', '_', lc_N)
        sso@project.name  <- gsub(' ', '_', sso@project.name)
        sso@project.name  <- gsub('\\.', '_', sso@project.name)
        lc_listR <- f_sc_degs(sso, lc_groupN, lc_listR)
        
    }
    sampleN <- gsub('/', '_', sampleN)
    sampleN <- gsub(' ', '_', sampleN)
    sampleN <- gsub('\\.', '_', sampleN)
    f_mkdir_s(sampleN)
    f_xlsx_output(paste0(sampleN, '_sObject_markers.xlsx'), lc_listR$M)
    f_xlsx_output(paste0(sampleN, '_significant_markers.xlsx'), lc_listR$SM)
    f_ppt_output(paste0(sampleN, '_Heatmap.pptx'), lc_listR$P)
    f_mkdir_e()
    return(lc_listR)
}
```

## 第五步 构造概述函数

```r
f_sc_DoHeatmap_null_summary <- function(lc_SM){
    rN <- names(lc_SM)
    cN <- NULL
    for(rNo in rN){
        SMo <- lc_SM[[rNo]]
        cN <- c(cN, as.character(SMo$cluster))
    }
    cN <- unique(cN)
    dD <- matrix(nrow=length(rN),ncol=length(cN),data = 0) 
    rownames(dD) <- rN
    colnames(dD) <- cN
    dD
}

f_sc_DoHeatmap_summary <- function(lc_SM){
    dD <- f_sc_DoHeatmap_null_summary(lc_SM)
    for(rNo in names(lc_SM)){
        SMo <- lc_SM[[rNo]]
        SMo <- SMo$cluster
        for(cNo in SMo){
            dD[rNo, cNo] <- dD[rNo, cNo] + 1
        }
    }
    dD
}
```

## 第六步 输出结果

```r
fuck <- SplitObject(scRNA_split, split.by = 'orig.ident')
sMa <- list()
for(lc_N in names(fuck)){
    fucko <- fuck[[lc_N]]
    fucko <- f_sc_DoHeatmap_sp(fucko, 'hM1_hmca_class', 'Region', lc_N)
    fucko <- f_sc_DoHeatmap_summary(fucko$SM)
    sMa[[lc_N]] <- data.frame(fucko)
}
f_xlsx_output('sc_DoHeatmap_summary.xlsx', sMa)
```