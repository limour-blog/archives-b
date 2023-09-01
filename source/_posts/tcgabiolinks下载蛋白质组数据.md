---
title: TCGAbiolinks下载蛋白质组数据
tags: []
id: '2020'
categories:
  - - uncategorized
date: 2022-09-10 15:44:13
---

之前[通过tcpa下载过蛋白数据](https://occdn.limour.top/2300.html)，而[TCGAbiolinks也有下载蛋白质组学数据的示例](https://bioconductor.org/packages/release/bioc/vignettes/TCGAbiolinks/inst/doc/download_prepare.html)，后者看上去更全面一点。

## 下载数据

```R
library(TCGAbiolinks)
query.rppa <- GDCquery(
    project = "TCGA-PRAD", 
    data.category = "Proteome Profiling",
    data.type = "Protein Expression Quantification"
)
GDCdownload(query.rppa) 
rppa <- GDCprepare(query.rppa)
saveRDS(rppa, 'PRAD_rppa.rds')
```

## 清洗数据

```R
pMiss <- function(x){round(sum(is.na(x))/length(x),3)}
rppa <- rppa[apply(rppa, 1, pMiss) < 0.05, ]
rppa <- rppa[, apply(rppa, 2, pMiss) < 0.05]
sum(is.na(rppa))
rowInfo <- rppa[1:5]
rppa <- rppa[-(1:5)]
rppa <- rppa[, substr(colnames(rppa), 14, 16) == "01A"]
colnames(rppa) <- substr(colnames(rppa),1,12)
group <- readRDS('../fig5/tcga.predict.rds')
gRow <- intersect(colnames(rppa), rownames(group))
group <- group[gRow,]
rppa <- rppa[, colnames(rppa) %in% gRow]
Ct1 <- rownames(group)[group$group == 'Low Risk']
Tt1 <- rownames(group)[group$group == 'High Risk']
```

## 计算差异蛋白

[f\_DE\_limma](https://occdn.limour.top/2171.html)

```R
r1 <- f_DE_limma(rppa, rowInfo, Ct1, Tt1, trend=F)
rownames(rppa) <- rowInfo$AGID
save(r1, rppa, file = 'PRAD_TCPA_DE.rdata')
```