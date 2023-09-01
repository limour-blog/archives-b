---
title: 将多数据集转化成无依赖的数据格式，进行单变量COXPH，并进行RRA整合
tags: []
id: '1953'
categories:
  - - uncategorized
date: 2022-08-08 09:53:15
---

## PCaDB

[f\_dedup\_IQR](https://occdn.limour.top/2157.html)

```R
ano <- readRDS('../../DEG/PCaDB/PCaDB_Gene_Annotation.RDS')
tmp <- readRDS('../../DEG/PCaDB/CPC-Gene_eSet.RDS')
tmp
clinical <- tmp@phenoData@data
clinical <- clinical[!is.na(clinical$bcr_status),]
sum(clinical$bcr_status)
sum(clinical$time_to_bcr)
expr <- (exprs(tmp))
expr <- f_dedup_IQR(as.data.frame(expr), ano[rownames(expr),]$gene_name)
expr <- scale(t(expr))
expr <- as.data.frame(expr[,!is.na(colSums(expr))])
expr
clinical <- clinical[rownames(clinical) %in% rownames(expr),]
sum(clinical$bcr_status)
sum(clinical$time_to_bcr)
clinical
expr <- expr[rownames(clinical),]
expr
clinical$pfs_status <- clinical$bcr_status
clinical$pfs_time <-  clinical$time_to_bcr
data <- list()
data$data <- expr
data$meta  <- clinical
saveRDS(data, 'CPC-Gene.rds')
```

## TCGA

[f\_counts2TMM](https://occdn.limour.top/2159.html)

```R
tmp <- load('../../DEG/TCGA/PRAD_tp.rda')
counts <- data@assays@data$unstranded
colnames(counts) <- data@colData$patient
counts <- counts[,f_rm_duplicated(colnames(counts))]
geneInfo <- as.data.frame(data@rowRanges)[c('gene_id','gene_type','gene_name')]
counts <- f_dedup_IQR(as.data.frame(counts), geneInfo$gene_name)
counts
counts <- f_counts2TMM(counts)
counts <- log2(counts + 1)
counts <- scale(t(counts))
counts <- as.data.frame(counts[,!is.na(colSums(counts))])
counts
clinical <- readRDS('../../DEG/TCGA/clinical/TCGA_PRAD_with_ICGC.rds')
clinical <- clinical[rownames(clinical) %in% rownames(counts),]
sum(clinical$new_dcf_status)
sum(clinical$new_dcf_time)
clinical
counts <- counts[rownames(clinical),]
counts
clinical$pfs_status <- clinical$new_dcf_status
clinical$pfs_time <- (clinical$new_dcf_time / 365)*12
data <- list()
data$data <- expr
data$meta  <- clinical
saveRDS(data, 'PRAD.rds')
```

## train\_sets

```R
train_sets <- list()
train_sets$CPGEA <- readRDS('data/CPGEA.rds')
train_sets$PRAD <- readRDS('data/PRAD.rds')
train_sets$mCPRC <- readRDS('data/mCRPC.rds')
train_sets$DKFZ <- readRDS('data/DKFZ.rds')
train_sets$GSE54460 <- readRDS('data/GSE54460.rds')
saveRDS(train_sets, 'train_sets.rds')
```

## 单变量COXPH

```R
library("survival")
library("survminer")
f_DEG_coxph_oneGene <- function(trainSet, geneN){
    if(!(geneN %in% colnames(trainSet$data))){
        return(data.frame(row.names = geneN, mean=NA, lower=NA, upper=NA, Pvalue=1, VarName=geneN))
    }
    df <- cbind(trainSet$data[geneN],trainSet$meta[c('pfs_status', 'pfs_time')])
    coxmf <- paste0("Surv(pfs_time, pfs_status==1)~", '`',geneN,'`')
#     print(coxmf)
    res.cox <- coxph(formula(coxmf), data =  df)
    tmp_res <- summary(res.cox)
    res <- as.data.frame(tmp_res$conf.int)
    res <- res[,c(1,3,4)]
    colnames(res) <- c('mean', 'lower', 'upper')
    res[['Pvalue']] <- tmp_res$coefficients[,'Pr(>z)']
    res[['VarName']] <- rownames(res)
    res
}
f_DEG_coxph <- function(trainSet, geneList){
    res <- NULL
    for(gene in geneList){
        res <- rbind(res,f_DEG_coxph_oneGene(trainSet, gene))
    }
    res
}
f_DEG_coxph_Sets <- function(trainSets, geneList){
    res <- list()
    for(Name in names(trainSets)){
        res[[Name]] <- f_DEG_coxph(trainSets[[Name]], geneList)
    }
    res
}
 
DEG <- readRDS('../DEG/TCGA/mCRPC_DEG.rds')
DEG <- subset(DEG, padj < 0.05 & abs(log2FoldChange)>2 & !grepl('pseudogene', gene_type))
train_sets <- readRDS('train_sets.rds')
tmp <- f_DEG_coxph_Sets(train_sets, unique(DEG$gene_name))
saveRDS(tmp, 'mCRPC.rds')
```

## RRA聚合

```R
tmp <- readRDS('mCRPC.rds')
require(RobustRankAggreg)
f_dflist_RRA <- function(dflist, nameN, N, orderN, decreasing=T){
    res <- list()
    for (name in names(dflist)){
        tmp <- dflist[[name]]
        if (nrow(tmp) == 1){
            res[[name]] <- tmp[1, nameN]
        }else if(nrow(tmp) >=2){
            res[[name]] <- tmp[order(tmp[[orderN]],decreasing = decreasing), nameN]
        }
    }
    if(length(res) < 1){ return(NULL)}
    aggregateRanks(glist = res, N = N)
}
tmp$CPGEA
r <- lapply(tmp, FUN = function(x){subset(x, Pvalue<0.05 & mean > 1)})
r <- f_dflist_RRA(r, 'VarName', nrow(tmp$CPGEA), 'mean')
r <- subset(r, Score<0.05)
r_up <- r
r <- lapply(tmp, FUN = function(x){subset(x, Pvalue<0.05 & mean < 1)})
r <- f_dflist_RRA(r, 'VarName', nrow(tmp$CPGEA), 'mean', decreasing = F)
r <- subset(r, Score<0.05)
r_down <- r
 
r_up$Score <- log1p(1/r_up$Score)
r_down$Score <- log1p(1/r_down$Score)
r_down$Score <- -r_down$Score
r <- rbind(r_up,r_down)
r <- r[order(r$Score, decreasing = T),]
r
saveRDS(r, 'mCRPC_RRA.rds')
```