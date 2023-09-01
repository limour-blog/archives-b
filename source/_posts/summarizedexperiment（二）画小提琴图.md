---
title: SummarizedExperiment（二）画小提琴图
tags: []
id: '1644'
categories:
  - - R绘图奇技淫巧
date: 2022-02-16 00:09:24
---

[https://www.jianshu.com/p/81f7d8ffe647](https://www.jianshu.com/p/81f7d8ffe647)

```r
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
require(ggplot2)
f_SCE_VlnBoxPlot <- function(df, geneN, groupN='groupN'){
    p <- ggplot(df, aes(x=!!sym(groupN), y=value, fill= !!sym(groupN), alpha = 0.618))
    p <- p + theme_bw() + theme (legend.position = "none") 
    p <- p + geom_violin() # 绘制小提琴图
#     p <- p + stat_ydensity(trim = TRUE, scale = 'width', adjust = 1) # 绘制小提琴图
    p <- p + geom_boxplot(width=0.618) # 绘制箱型图
    p <- p + stat_summary(fun="mean",geom="point",color='white') # 添加均值点
    p <- p + labs(x=NULL, y=NULL) # 删除xy轴标题
    p <- p + labs(title=geneN) + theme(plot.title = element_text(hjust = 0.5))
    p <- p + theme(axis.text.x=element_text(hjust = 1, angle = 45))
    p
}
```

```r
f_SCE_VlnBoxPlot(f_SCE_VlnBoxPlot_gD('GAPDH', prad_p=prad_p, prad_n=prad_n, crpc=crpc), 'GAPDH log1p(TPM)')
```