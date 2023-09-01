---
title: R语言-有序分类logistic回归
tags: []
id: '1688'
categories:
  - - 巨塔砖石
date: 2022-02-18 16:16:05
---

## 第一步 获得数据

```r
f_TCGA_gleason_grade <- function(primary_gleason_grade, secondary_gleason_grade){
    primary_gleason_grade <- as.numeric(unlist(data.frame(strsplit(primary_gleason_grade, ' '))[2,]))
    secondary_gleason_grade <- as.numeric(unlist(data.frame(strsplit(secondary_gleason_grade, ' '))[2,]))
    primary_gleason_grade + secondary_gleason_grade
}
f_rank_transformation_o <- function(lo){
    lo <- unlist(lo)
    idx <- order(lo, decreasing = F)
    idm <- as.data.frame(table(lo))
    idm <- idm[idm$Freq>1, ]
    idm <- idm[[1]]
    for(i in idm){
        ii <- (lo == i)
        idx[ii] <- mean(idx[ii])
    }
    names(idx) <- names(lo)
    idx
}
f_SCE_VlnBoxPlot_gD <- function(featureN, assayN='TPM', rank_trans=F, rmZero=NULL, log_trans=T, ...){
    sceList <- list(...)
    df <- data.frame()
    for(name in names(sceList)){
        sce <- sceList[[name]]
        idx <- rowRanges(sce)$external_gene_name == featureN
        tmp <- assay(sce[idx, ], assayN)
        tmp <- data.frame(groupN=name, value=tmp[1,])
        df <-  rbind(df, tmp)
    }
    if(log_trans){
        df$value <- log1p(df$value)
    }
    if(rank_trans){
        df$value <- f_rank_transformation_o(df$value)
        df
    }else{
        if(!is.null(rmZero)){
            df[df$value>rmZero, ]
        }else{df}
    }   
}
```

```r
d <- f_SCE_VlnBoxPlot_gD('SMC4', prad_p=prad_p)
d['gleason'] <- f_TCGA_gleason_grade(prad_p$primary_gleason_grade, prad_p$secondary_gleason_grade)
saveRDS(d, 'prad_p_gleason.rds')
```

## 第二步 更换内核进行分析

*   [通过conda安装纯净环境的Logistic分析包](https://limour.top/1690.html)

```r
require(foreign)
require(ggplot2)
require(MASS)
require(Hmisc)
require(reshape2)

dat <- readRDS('prad_p_gleason.rds')
```

```r
m <- polr(factor(gleason) ~ value, data = dat, Hess = TRUE)
m
drop1(m,test="Chi") 
(ci <- confint(m))
exp(cbind(OR = coef(m), ci))
```