---
title: COXPH的变量筛选
tags: []
id: '1961'
categories:
  - - uncategorized
date: 2022-08-10 02:36:32
---

返工的血泪教训，单因素COXPH做完显著是不够的，此时应该再来一次多因素COXPH，剔除那些不具有独立预后能力的变量。

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
f_DEG_coxph_GeneList <- function(dat, geneNs){
    if(length(geneNs) == 1){
        return(f_DEG_coxph_oneGene(dat,geneNs))
    }
    y <- Surv(round(dat$meta$pfs_time,2), dat$meta$pfs_status == 1)
    y <- data.matrix(y)
    x <- dat$data[,geneNs]
    x <- as.matrix(x)
    df <- cbind(x,y)
    df <- as.data.frame(df)
    coxmf <- paste0("Surv(time, status==1)~", paste(paste('`',geneNs,'`',sep = ''), collapse = '+'))
    res.cox <- coxph(formula(coxmf), data =  df)
    tmp_res <- summary(res.cox)
    res <- as.data.frame(tmp_res$conf.int)
    res <- res[,c(1,3,4)]
    colnames(res) <- c('mean', 'lower', 'upper')
    res[['Pvalue']] <- tmp_res$coefficients[,'Pr(>z)']
    res[['VarName']] <- rownames(res)
    res
}
f_DEG_coxph <- function(trainSet, geneList, FPv=0.05, strict=T){
    res <- NULL
    for(gene in geneList){
        res <- rbind(res,f_DEG_coxph_oneGene(trainSet, gene))
    }
    if(!strict){
        return(res)
    }
    res <- subset(res, Pvalue<FPv)$VarName
    if(length(res) == 0){
        return(data.frame(mean=NA, lower=NA, upper=NA, Pvalue=1, VarName=NA))
    }
    res <- f_DEG_coxph_GeneList(trainSet, res)
    res[order(res$Pvalue),]
}
f_DEG_coxph_Sets <- function(trainSets, geneList, FPv=0.05, strict=T){
    res <- list()
    for(Name in names(trainSets)){
        res[[Name]] <- f_DEG_coxph(trainSets[[Name]], geneList, FPv = FPv, strict = strict)
    }
    res
}
x <- readRDS('../A_ref_A_fiig.2_A/hub.rds')
train_sets <- readRDS('../../../COXPH/train_sets.rds')
tmp <- f_DEG_coxph_Sets(train_sets, x)
```