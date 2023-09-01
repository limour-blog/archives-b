---
title: counts转fpkm，fpkm转tpm
tags: []
id: '1918'
categories:
  - - uncategorized
date: 2022-07-19 00:08:11
---

不知道为什么counts转fpkm的结果和tcga的结果对不上，可能里面的colSums用的是reads mapped to all protein-coding regions？没有进行进一步的尝试。fpkm转tpm的结果和tcga的结果是一致的。

```R
f_counts2fpkm <- function(counts, gene_lengths){ # 确保记录已经对齐
    lc_rpk <- (counts/gene_lengths) * 10^3
    lc_fpkm <- (lc_rpk/colSums(counts)) * 10^6
    lc_fpkm
}
f_fpkmToTpm <- function(l_e){
     apply(l_e,2,function(fpkm){exp(log(fpkm) - log(sum(fpkm)) + log(1e6))})
}
```