---
title: GEOquery (二) raw_counts转fpkm
tags: []
id: '1683'
categories:
  - - 组织测序分析
date: 2022-02-18 13:15:09
---

## 第一步 下载基因长度信息

[https://file-cdn.limour.top/bix/22.02.18.tcga\_rowRanges.csv.gz](https://file-cdn.limour.top/bix/22.02.18.tcga_rowRanges.csv.gz)

## 第二步 读取基因长度信息

```r
grg <- read.csv('22.02.18.tcga_rowRanges.csv')
```

## 第三步 构建counts转fpkm函数

```r
f_counts2fpkm <- function(ct, grg, gidN='ensembl_gene_id', rowN=NULL){
    if(is.null(rowN)){
        rowN <- rownames(ct)
    }
    grg <- grg[c(gidN, 'width')]
    grg['width'] <- grg['width']/1000 # 转kb
    idx <- match(rowN, grg[[1]])
    idx_f <- !is.na(idx) # 去除没有记录的基因
    ct <- ct[idx_f,] 
    idx <- idx[idx_f]
    grg <- grg[idx,] # 记录对齐
    lc_rpk <- ct/grg[[2]]
    lc_fpkm <- t(t(lc_rpk)/colSums(ct) * 10^6)
    lc_fpkm
}
```

## 第四步 计算并转换基因名

```r
f_id2name <- function(lc_exp, lc_db){
    if(!is.data.frame(lc_db)){
        lc_ids <- toTable(lc_db)
    }else{
        lc_ids <- lc_db
    }
    res <- lc_exp[rownames(lc_exp) %in% lc_ids[[1]],]
    lc_ids=lc_ids[match(rownames(res),lc_ids[[1]]),]
    lc_tmp = by(res,
         lc_ids[[2]],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res = res[rownames(res) %in% lc_probes,]
    rownames(res)=lc_ids[match(rownames(res),lc_ids[[1]]),2]
    res
}
```

```r
d2 <- f_counts2fpkm(d, grg)
d2 <- f_id2name(d2, grg[c('ensembl_gene_id', 'external_gene_name')])
```