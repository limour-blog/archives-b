---
title: SummarizedExperiment（三）counts转dds
tags: []
id: '1647'
categories:
  - - 组织测序分析
date: 2022-02-16 11:01:47
---

```r
f_SCE_2DDS <- function(assayN='HTSeq - Counts', ...){
    sceList <- list(...)
    condition <- NULL
    countData <- NULL
    colData <- NULL
    for(name in names(sceList)){
        sce <- sceList[[name]]
        tmp <- assay(sce, assayN)
        rownames(tmp) <- rowRanges(sce)$external_gene_name
        if(is.null(countData)){
            countData <- tmp
        }else{
            countData <- cbind(countData, tmp)
        }
        condition <- c(condition, rep(name, ncol(tmp)))
    }
    condition <- factor(condition)
    colData <- data.frame(row.names = colnames(countData), condition)
    DESeqDataSetFromMatrix(countData = countData, colData = colData, design = ~condition)
}
```

```r
f_SCE_VlnBoxPlot_dds <- function(featureN, dds){
    df <- as.data.frame(t(assay(dds[featureN, ])))
    df[['groupN']] <- dds@colData$condition
    colnames(df) <- c('value', 'groupN')
    df
}
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