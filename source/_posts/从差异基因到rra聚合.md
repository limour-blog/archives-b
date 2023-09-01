---
title: 从差异基因到RRA聚合
tags: []
id: '1925'
categories:
  - - uncategorized
date: 2022-07-22 03:11:31
---

通过[比对](https://occdn.limour.top/1934.html)，我们得到了counts矩阵，接下来可以进行DEGs分析。此时如果我们有多组之间的对比，则可以使用[RRA算法](https://occdn.limour.top/1658.html)来聚合我们的结果。[RRA的安装过程见此](https://occdn.limour.top/1653.html)。

## 第一步，多组差异基因分析

```R
library(DESeq2)
count_all <- read.csv("~/upload/zl_liu/star_data/yyz_01/yyz_01.csv",header=TRUE)
count_all
cts_b <- count_all[ ,c(-1,-2,-3)]
rownames(cts_b) <- count_all$ID
keep <- rowSums(cts_b) > ncol(cts_b)
cts_b[keep,]
f_DESeq2 <- function(cts_bb, rowInfo, ControlN, TreatN, rm.NA=T){
    cts_b <- cts_bb[,c(ControlN, TreatN)]
    conditions <- factor(c(rep("Control",length(ControlN)), rep("Treat",length(TreatN)))) 
    colData_b <- data.frame(row.names = colnames(cts_b), conditions)
    print(colData_b)
    dds <- DESeqDataSetFromMatrix(countData = cts_b,
                              colData = colData_b,
                              design = ~ conditions)
    dds <- DESeq(dds)
    res <- results(dds)
    rres <- cbind(rowInfo, data.frame(res))
    if(rm.NA){rres <- rres[!is.na(rres$padj),]}
    rres <- rres[order(rres$log2FoldChange, decreasing = T),]
    saveRDS(rres, paste('DEGs', paste(TreatN, collapse = '_'), 'vs.', paste(ControlN, collapse = '_'), 'DESeq2.rds',sep = '_'))
    rres
}
Ct1 <- c('X1.control.DMSO', 'X2.control.DMSO')
Tt1 <- c('X1.OE.DMSO', 'X2.OE.DMSO')
Ct2 <- c('X1.OE.DMSO', 'X2.OE.DMSO')
Tt2 <- c('X1.OE.Enz', 'X2.OE.Enz')
Ct3 <- c('X1.control.Enz', 'X2.control.Enz')
Tt3 <- c('X1.OE.Enz', 'X2.OE.Enz')
Ct4 <- c('X1.CAF.DMSO', 'X2.CAF.DMSO')
Tt4 <- c('X1.CAF.Enz', 'X2.CAF.Enz')
r1 <- f_DESeq2(cts_b[keep,], count_all[keep,c(1,2,3)], Ct1, Tt1)
r2 <- f_DESeq2(cts_b[keep,], count_all[keep,c(1,2,3)], Ct2, Tt2)
r3 <- f_DESeq2(cts_b[keep,], count_all[keep,c(1,2,3)], Ct3, Tt3)
r4 <- f_DESeq2(cts_b[keep,], count_all[keep,c(1,2,3)], Ct4, Tt4)
```

## 第二步，RRA聚合差异结果

```R
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
rownames(count_all) <- count_all$ID
 
r <- list(r1=r1,r2=r2,r3=r3,r4=r4)
r <- lapply(r, FUN = function(x){subset(x, padj<0.05 & log2FoldChange > 1)})
r <- f_dflist_RRA(r, 'ID', sum(keep), 'log2FoldChange')
r <- cbind(count_all[r$Name,c(2,3)], r)
r
write.csv(r, 'RRA_up.csv')
 
r <- list(r1=r1,r2=r2,r3=r3,r4=r4)
r <- lapply(r, FUN = function(x){subset(x, padj<0.05 & log2FoldChange < -1)})
r <- f_dflist_RRA(r, 'ID', sum(keep), 'log2FoldChange', decreasing = F)
r <- cbind(count_all[r$Name,c(2,3)], r)
r
write.csv(r, 'RRA_down.csv')
```