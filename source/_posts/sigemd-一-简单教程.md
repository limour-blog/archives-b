---
title: SigEMD (一) 简单教程
tags: []
id: '1022'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-10-11 12:49:25
---

## 第一步 转换数据

```
library('biomaRt')
mart <- useDataset("hsapiens_gene_ensembl", useMart("ensembl"))
genes <- rownames(gene_len) 
G_list <- getBM(filters= "ensembl_gene_id", attributes= c("ensembl_gene_id","hgnc_symbol"),values=genes,mart= mart)
G_list <- subset(G_list, hgnc_symbol!="")‘
gene_len <- readRDS('exons_gene_lens.rds')

f_id2name_fuck <- function(lc_exp, lc_db){
    if(!is.data.frame(lc_db)){
        lc_ids <- toTable(lc_db)
    }else{
        lc_ids <- lc_db
    }
    res_n <- rownames(lc_exp)
    res_n <- res_n[res_n %in% lc_ids[[1]]]
    res <- lc_exp[res_n,]
    if(!is.data.frame(res)){res=data.frame(row.names = res_n, res)}
    lc_ids=lc_ids[match(rownames(res),lc_ids[[1]]),]
    lc_tmp = by(res,
         lc_ids[[2]],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res_n <- rownames(res)
    res_n <- res_n[res_n %in% lc_probes]
    res = res[res_n,]
    if(!is.data.frame(res)){res=data.frame(row.names = res_n, res)}  
    rownames(res)=lc_ids[match(rownames(res),lc_ids[[1]]),2]
    res
}

gene_len <- f_id2name_fuck(gene_len,G_list)

```

```
a <- f_seurat2myd(scRNA, 'group', 'batch', lc_tpm = F)
```

## 第二步 去除批次效应

```
library(sva)
library(limma)
f_removeBE <- function(lc_dat, lc_batch, lc_g, use_ComBat=F){
    if(use_ComBat){
        if(use_ComBat == 2){
            res <- ComBat_seq(counts = lc_dat, batch = lc_batch, group = lc_g, full_mod = T)
        }else{
            lc_mod <- model.matrix(~as.factor(lc_g))
            res <- ComBat(dat = lc_dat, batch = as.factor(lc_batch), mod = lc_mod)
        }
    }else{
        lc_mod <- model.matrix(~0+as.factor(lc_g))
        res <- removeBatchEffect(lc_dat, batch = as.factor(lc_batch), design = lc_mod)
    }
    res
}

```

```
a$c <- f_removeBE(a$c,a$b,a$g,use_ComBat = 2)
```

## 第三步 使用SigEMD

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-2.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-2.webp)

emm，前面都不用做了

```
mkdir SigEMD
cd SigEMD/
wget https://github.com/NabaviLab/SigEMD/archive/refs/heads/master.zip
unzip master.zip
```

```
setwd("~/zlliu/R_data/TCGA/SigEMD//SigEMD-master")
# options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
# BiocManager::install("lars", update=F)
# SigEMD is based on R package "aod","arm","fdrtool","lars"
library(aod)
library(arm)
library(fdrtool)
library(lars)
source("FunImpute.R")
source("SigEMDHur.R")
source("SigEMDnonHur.R")
source("plot_sig.R")

data <- dataclean(scRNA[['RNA']]@data)
databinary<- databin(data)
condition <- unlist(scRNA[['group']])
names(condition) <- colnames(data)
Hur_gene<- idfyImpgene(data,databinary,condition) # 耗时较久
genes_use<- idfyUsegene(data,databinary,condition,ratio = 0.5)
data <- data.frame(data)
gc()
datimp <- FunImpute(object = data, genes_use = (genes_use), genes_fit = (Hur_gene),dcorgene = NULL) # 久到无法接受
data<-datimp$alldat 
results<- calculate_single(data =  data,condition =  condition,Hur_gene = Hur_gene, binSize=0.2,nperm=5)
emd<- results$emdall
head(emd)

```

## 第四步 使用FindAllMarkers

```
Idents(scRNA) <- scRNA[['group']]
all_markers <- FindAllMarkers(scRNA, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)

```