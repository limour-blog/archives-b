---
title: 不同批次但未记录批次时使用DESeq2的方法
tags: []
id: '1954'
categories:
  - - uncategorized
date: 2022-08-08 16:56:44
---

## 准备数据

```R
library(SummarizedExperiment)
tmp <- load('PRAD_tp.rda')
counts <- data@assays@data$unstranded
colnames(counts) <- data@colData$patient
f_rm_duplicated <- function(NameL, reverse=F){
    tmp <- data.frame(table(NameL))
    if(reverse){
        tmp <- tmp$NameL[tmp$Freq > 1]
    }else{
        tmp <- tmp$NameL[tmp$Freq == 1]
    }
    which(NameL %in% as.character(tmp))
}
counts <- counts[,f_rm_duplicated(colnames(counts))]
geneInfo <- as.data.frame(data@rowRanges)[c('gene_id','gene_type','gene_name')]
tmp <- load('mCRPC.rda')
geneInfo2 <- as.data.frame(data@rowRanges)[c('gene_id','gene_type','gene_name')]
all(geneInfo2$gene_id == geneInfo$gene_id)
counts2 <- data@assays@data$unstranded
colnames(counts2) <- data@colData$bcr_patient_barcode
counts2 <- counts2[,f_rm_duplicated(colnames(counts2))]
colnames(counts) <- paste('N', colnames(counts), sep = '_')
cts_b <- as.data.frame(cbind(counts2,counts))
rownames(geneInfo) <- NULL
keep <- (rowSums(counts) > ncol(counts)) & (rowSums(counts2) > ncol(counts2))
cts_b[keep,]
```

## 安装补充包

*   [conda activate tcga](https://occdn.limour.top/1653.html)
*   conda install -c bioconda bioconductor-sva -y

## 计算DEG

```R
library(DESeq2)
library(sva)
f_DESeq2_sva <- function(cts_bb, rowInfo, ControlN, TreatN, rm.NA=T){
    cts_b <- cts_bb[,c(ControlN, TreatN)]
    conditions <- factor(c(rep("Control",length(ControlN)), rep("Treat",length(TreatN)))) 
    colData_b <- data.frame(row.names = colnames(cts_b), conditions)
    ## 生物学差异的设计矩阵
    group <- as.factor(c(rep("Ctl",length(ControlN)), rep("Trt",length(TreatN))))
    mod1  <-  model.matrix(~group)
    print(mod1)
    ## 无差异的设计矩阵
    mod0  <-  cbind(mod1[,1])
    svseq  <-  svaseq(as.matrix(cts_b),mod1,mod0,n.sv=2) # Estimate batch with svaseq (unsupervised)
    colData_b$SVa <- svseq$sv[,1]
    colData_b$SVb <- svseq$sv[,2]
    print(colData_b)
    ddssva <- DESeqDataSetFromMatrix(countData = cts_b,
                          colData = colData_b,
                          design = ~ SVa + SVb + conditions)
    ddssva <- DESeq(ddssva)
    
    res <- results(ddssva, contrast=c("conditions","Treat","Control"))
    rres <- cbind(rowInfo, data.frame(res))
    if(rm.NA){rres <- rres[!is.na(rres$padj),]}
    rres <- rres[order(rres$log2FoldChange, decreasing = T),]
    # saveRDS(rres, paste('DEGs', paste(TreatN, collapse = '_'), 'vs.', paste(ControlN, collapse = '_'), 'DESeq2.rds',sep = '_'))
    rres
}
```

```R
Ct1 <- colnames(counts)
Tt1 <- colnames(counts2)
r1 <- f_DESeq2_sva(cts_b[keep,], geneInfo[keep,], Ct1, Tt1)
```