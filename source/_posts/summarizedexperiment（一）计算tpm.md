---
title: SummarizedExperiment（一）计算TPM
tags: []
id: '1642'
categories:
  - - 组织测序分析
date: 2022-02-15 22:02:26
---

```r
fpkmToTpm <- function(fpkm){
    exp(log(fpkm) - log(sum(fpkm)) + log(1e6))
}
f_fpkmToTpm <- function(l_e){
     apply(l_e,2,fpkmToTpm)
}
assay(sce, "TPM") <- f_fpkmToTpm(assay(sce, "HTSeq - FPKM"))
```