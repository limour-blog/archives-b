---
title: scran：使用fastMNN算法移除批次效应
tags: []
id: '1972'
categories:
  - - uncategorized
date: 2022-08-13 07:18:35
---

为了复现[这篇Nature Cell Biology上文章](https://www.nature.com/articles/s41556-020-00613-6)的结果，试试文章提供的代码

## 安装补充包

*   [conda activate seurat](https://occdn.limour.top/1534.html)
*   conda install -c bioconda bioconductor-scran -y
*   conda install -c bioconda bioconductor-scater -y
*   install.packages("BoutrosLab.plotting.general")
*   conda install -c bioconda bioconductor-batchelor -y
*   \# conda install -c bioconda bioconductor-biocparallel -y

## 读入UMI矩阵(已QC)

```R
scRNA <- read.table( gzfile("GSM4203181/GSM4203181_data.raw.matrix.txt.gz"), header = T, row.names = 1)
```

## 构建SingleCellExperiment

```R
library(scran);
library(Seurat);
library(scater);
library(batchelor);
sce <- SingleCellExperiment(list(counts=scRNA))
sce$batch <- paste0('sample',substr(colnames(sce),18,nchar(colnames(sce))))
by.lib <- split(seq_len(ncol(sce)), sce$batch);
cluster.id <- character(ncol(sce));
for (lib in names(by.lib)) { 
    print(lib)
    current <- by.lib[[lib]]
    cur.exprs <- realize(counts(sce)[,current]) # for speed; avoid multiple file reads here.
    ids <- quickCluster(cur.exprs, min.mean=0.1, method="igraph")
    cluster.id[current] <- paste0(lib, ".", ids)
}
sce <- computeSumFactors(sce, cluster=cluster.id, min.mean=0.1, max.cluster.size=3000);
sce <- logNormCounts(sce);
saveRDS(sce, 'clustered_normalized.rds');
```

## 运行MNN

```R
library(scran);
library(Seurat);
library(scater);
library(batchelor);
sce <- readRDS('clustered_normalized.rds')
batch <- as.numeric(as.factor(sce$batch));
count1 <- logcounts(sce)[, batch == 1];
count2 <- logcounts(sce)[, batch == 2];
count3 <- logcounts(sce)[, batch == 3];
count4 <- logcounts(sce)[, batch == 4];
count5 <- logcounts(sce)[, batch == 5];
count6 <- logcounts(sce)[, batch == 6];
count7 <- logcounts(sce)[, batch == 7];
count8 <- logcounts(sce)[, batch == 8];
count9 <- logcounts(sce)[, batch == 9];
count10 <- logcounts(sce)[, batch == 10];
count11 <- logcounts(sce)[, batch == 11];
count12 <- logcounts(sce)[, batch == 12];
count13 <- logcounts(sce)[, batch == 13, drop = FALSE];
original <- list(count1, count2, count3, count4, count5, count6, count7, count8,
    count9, count10, count11, count12, count13);
names(original) <- levels(as.factor(as.factor(sce$batch)))
gc()
system(paste("python3 /home/jovyan/upload/zl_liu/wecomchan.py", "'fastMNN_task start'"), intern = T)
out  <-  tryCatch({
      #正常的逻辑
    do.call(fastMNN, c(original, list(k = 20, d = 50, BPPARAM = BiocParallel::MulticoreParam(6))));
  }, warning = function(w) {
      #出现warning的处理逻辑
    w <- as.character(w)
    w <- gsub("[\r\n']", "", w)
    system(paste("python3 /home/jovyan/upload/zl_liu/wecomchan.py", paste0("'",w,"'")), intern = T)
    w
  }, error = function(e) {
      #出现error的处理逻辑
    e <- as.character(e)
    e <- gsub("[\r\n']", "", e)
    system(paste("python3 /home/jovyan/upload/zl_liu/wecomchan.py", paste0("'",e,"'")), intern = T)
    e
  }, finally = {
      #不管出现异常还是正常都会执行的代码模块，
      #一般用来处理清理操作，例如关闭连接资源等。
})
system(paste("python3 /home/jovyan/upload/zl_liu/wecomchan.py", "'fastMNN completed'"), intern = T)
```

## 创建sce

```R
combined <- do.call(cbind, original);
sce <- SingleCellExperiment(list(logcounts = combined));
reducedDim(sce, 'MNN') <- reducedDim(out, 'corrected')
sce$Batch <- paste0('sample',substr(colnames(sce),18,nchar(colnames(sce))))
gc()
saveRDS(sce, 'sce.rds')
```

## 合并sce到seurat

```R
library(scran);
library(Seurat);
library(scater);
library(batchelor);
scRNA <- read.table( gzfile("GSM4203181/GSM4203181_data.raw.matrix.txt.gz"), header = T, row.names = 1)
sce <- readRDS('sce.rds')
count.log <- logcounts(sce)
scRNA <- scRNA[rownames(count.log),colnames(count.log)]
tmp <- SingleCellExperiment(list(counts=scRNA, logcounts=count.log))
tmp$Batch <- sce$Batch
reducedDim(tmp, 'MNN') <- reducedDim(sce, 'MNN')
saveRDS(tmp, 'scRNA.rds')
as.Seurat(tmp)
```

## 画图看看效果

```R
sce <- runTSNE(sce, use_dimred="MNN");
plotTSNE(sce, colour_by="Batch")
```

![](https://img.limour.top/archives_2023/2022/08/13/62f7500076865.webp)

效果稀碎。。。