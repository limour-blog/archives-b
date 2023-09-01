---
title: Counts矩阵的标准化方法：TMM和VST、RLOG
tags: []
id: '1934'
categories:
  - - uncategorized
date: 2022-07-27 12:03:03
---

*   [TMM](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2010-11-3-r25)：The Trimmed Mean of M value by [edgeR](https://www.jianshu.com/p/0e1ad0cc4ce6)
*   [VST](http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html)：The variance stabilizing transformation by [DESeq2](https://blog.csdn.net/leianuo123/article/details/112424578)
*   [RLOG](http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html)：The regularized-logarithm transformation by [DESeq2](https://blog.csdn.net/leianuo123/article/details/112424578)

Counts矩阵来源于[STAR匹配得到的结果](https://occdn.limour.top/1934.html)：`df <- read.csv('GSE123379.csv', row.names = 1)`

## 安装补充包

*   [conda activate tcga](https://occdn.limour.top/1653.html)
*   conda install -c bioconda bioconductor-edger -y

## TMM方法

```R
f_counts2TMM <- function(countsMatrix){
    require(edgeR)
    TMM <- DGEList(counts = countsMatrix)
    TMM <- calcNormFactors(TMM, method = 'TMM')
    cpm(TMM, normalized.lib.sizes = TRUE, log=F)
}
countsMatrix <- df[-(1:3)]
TMM <- f_counts2TMM(countsMatrix)
TMM
```

## VST方法

```R
f_counts2VST <- function(countsMatrix){
    require(DESeq2)
    conditions <- factor(rep("Control",ncol(countsMatrix)))
    colData_b <- data.frame(row.names = colnames(countsMatrix), conditions)
    dds <- DESeqDataSetFromMatrix(countData = countsMatrix,
                              colData = colData_b,
                              design = ~ 1)
    vsd <- vst(object=dds, blind=T) 
    assay(vsd)
}
countsMatrix <- df[-(1:3)]
VST <- f_counts2VST(countsMatrix)
VST
```

## RLOG方法

```R
f_counts2RLOG <- function(countsMatrix){
    require(DESeq2)
    conditions <- factor(rep("Control",ncol(countsMatrix)))
    colData_b <- data.frame(row.names = colnames(countsMatrix), conditions)
    dds <- DESeqDataSetFromMatrix(countData = countsMatrix,
                              colData = colData_b,
                              design = ~ 1)
    rld  <- rlog(object=dds, blind=T) 
    assay(rld)
}
countsMatrix <- df[-(1:3)]
RLOG <- f_counts2RLOG(countsMatrix)
RLOG
```