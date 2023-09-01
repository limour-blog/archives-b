---
title: Seurat (九) hclust聚类分析 (二) by 细胞比例
tags: []
id: '1352'
categories:
  - - 单细胞下游分析
date: 2021-12-17 14:07:03
---

## 第一步 读入数据

[Seurat (十) 同群细胞在不同脑区的DEGs （二）第二步](https://limour.top/1334.html)

## 第二步 构造相关计算函数

```r
f_ggplot2_ti <- function(p, title, subtitle=''){
    if (subtitle == ''){
        (p + ggtitle(title) + theme(plot.title = element_text(hjust = 0.5)))
    }else{
        (p + ggtitle(title, subtitle=subtitle) + 
         theme(plot.title = element_text(hjust = 0.5), plot.subtitle = element_text(hjust = 0.5)))
    }
}
 
library(ggdendro)
f_ggdendrogram <- function(hc){
    ggdendrogram(hc$h, rotate = T, size = 3)+theme(axis.text = element_text(size=14,face = "bold"))
}
 
f_hc <- function(lc_counts){
    d <- dist(t(lc_counts))
    h <- hclust(d)
    list(d=d, h=h)
}
 
f_hc_cop <- function(hc){
    cop <- cophenetic(hc$h)
    cor(cop, hc$d)
}
 
 
f_cluster_averages <- function(lc_scRNA, lc_metaN='ident'){
    # 切分出Clusters
    lc_clusters <- SplitObject(lc_scRNA, split.by = lc_metaN)
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- lc_clusters[[lc_i]][[lc_clusters[[lc_i]]@active.assay]]@scale.data
    }
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- apply(lc_clusters[[lc_i]],1,mean)
    }
    lc_clusters <- data.frame(lc_clusters)
    scale(lc_clusters)
}

f_br_cluster_f <- function(sObject, lc_groupN){
    lc_filter <- unlist(unique(sObject[[lc_groupN]]))
    lc_filter <- lc_filter[!is.na(lc_filter)]
    lc_filter
}

f_br_cluster <- function(sObject, lc_groupN, lc_labelN, lc_prop = F){
    lc_all <- unique(sObject[[lc_labelN]])
    rownames(lc_all) <- lc_all[[1]]
    colnames(lc_all) <- "CB"
#     lc_tp  <- SplitObject(subset(x = sObject, !!sym(lc_groupN)%in%f_br_cluster_f(sObject, lc_groupN)), split.by = lc_groupN)
    lc_tp  <- SplitObject(sObject, split.by = lc_groupN)
    for(lc_i in 1:length(lc_tp)){
        if(lc_prop){
            lc_tp[[lc_i]] <- prop.table(table(lc_tp[[lc_i]][[lc_labelN]]))
        }else{
            lc_tp[[lc_i]] <- table(lc_tp[[lc_i]][[lc_labelN]])
        }
    }
    for(lc_name in names(lc_tp)){
        lc_all[[lc_name]] = 0
        lc_all[names(lc_tp[[lc_name]]), lc_name] = lc_tp[[lc_name]]
    }
    lc_all[,-1]
}
```

*   相关性验证的代码

```r
library(Hmisc)
library(ggcorrplot)
f_corrplot <- function(lc_scdata){
    lc_cor <- rcorr(lc_scdata)
    lc_cor$P[is.na(lc_cor$P)]=0
    ggcorrplot(lc_cor$r, type="full",hc.order = T, lab = T, p.mat = lc_cor$P)
}
```

## 第三步 （可选）筛选感兴趣的细胞群

```r
n_ExN <- c('L4 IT','L5 IT','L5 ET','IT','L6b','L5/6 IT Car3','L6 IT','L2/3 IT','L5/6 NP','L6 IT Car3','L6 CT')
 
n_InN <- c('Lamp5','Pvalb','Sst','Vip','Sncg')
 
n_NoN <- c('Astro','PAX6','Endo','Micro-PVM','OPC','Oligo','Pericyte','VLMC')
 
n_groups <- list(NoN=n_NoN, ExN=n_ExN, InN=n_InN)
 
f_listUpdateRe <- function(lc_obj, lc_bool, lc_item){
    lc_obj[lc_bool] <- rep(lc_item,times=sum(lc_bool))
    lc_obj
}
 
f_grouplabel <- function(lc_meta.data, lc_groups){
    res <- lc_meta.data[[1]]
    for(lc_g in names(lc_groups)){
        lc_bool = (res %in% lc_groups[[lc_g]])
        for(c_n in colnames(lc_meta.data)){
            lc_bool = lc_bool  (lc_meta.data[[c_n]] %in% lc_groups[[lc_g]])
        }
        res <- f_listUpdateRe(res, lc_bool, lc_g)
    }
    names(res) <- rownames(lc_meta.data)
    res
}

scRNA_split[['n_groups']] <- f_grouplabel(scRNA_split[[c("hM1_hmca_class")]], n_groups)

sc_Neuron  <- subset(x = scRNA_split, n_groups %in% c("InN", "ExN"))
```

## 第四步 进行计算

*   by gene

```r
test_N <- SplitObject(sc_Neuron, split.by = 'orig.ident')
test_N[['all']] <- sc_Neuron

listPN <- list()
for (n in names(test_N)){
    test = f_cluster_averages(test_N[[n]], "Region")
    hc <- f_hc(test)
    cop <- paste(n, hc %>% f_hc_cop())
    print(cop)
    listPN[[n]] <- (f_hc(test) %>% f_ggdendrogram() %>% f_ggplot2_ti(paste0('hclust by gene in ', n), subtitle = cop)) +
    (f_corrplot(test) %>% f_ggplot2_ti(paste0('cor by gene in ', n)))
}
```

*   by proportion of cells

```r
test_s <- SplitObject(scRNA_split, split.by = 'orig.ident')
test_s[['all']] <- scRNA_split

listP <- list()
for (n in names(test_s)){
    test = as.matrix(f_br_cluster(test_s[[n]], 'Region', 'hM1_hmca_class', lc_prop = T))
    hc <- f_hc(test)
    cop <- paste(n, hc %>% f_hc_cop())
    print(cop)
    listP[[n]] <- (f_hc(test) %>% f_ggdendrogram() %>% f_ggplot2_ti(paste0('hclust by proportion of cells in ', n), subtitle = cop)) +
    (f_corrplot(test) %>% f_ggplot2_ti(paste0('cor by proportion of cells of ', n)))
}
```

## 第五步 输出计算结果

```r
# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"
 
# 配置结果保存路径
output_path = "~/zlliu/R_output/21.12.17.pcell_hclust"
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

library(export)
f_ppt_output <- function(fileName, image, lc_aspectr=1.5){
    for (p in image){
        graph2ppt(x=p, file=fileName, width=12, aspectr=lc_aspectr, append = TRUE, vector.graphic=F)
    }
}

f_ppt_output('split.pptx', listP, lc_aspectr = 2)
f_ppt_output('sc_Neuron.pptx', listPN, lc_aspectr = 2)
```