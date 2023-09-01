---
title: TCGAbiolinks下载maf数据
tags: []
id: '2008'
categories:
  - - uncategorized
date: 2022-09-06 09:04:57
---

## 下载数据

*   ~/dev/xray/xray -c ~/etc/xui2.json &

```R
library(TCGAbiolinks)
Sys.setenv("http_proxy"="http://127.0.0.1:20809")
Sys.setenv("https_proxy"="http://127.0.0.1:20809")
PRAD <- GDCquery(project = 'TCGA-PRAD',
                data.category = "Simple Nucleotide Variation",
                access = "open", 
                legacy = FALSE, 
                data.type = "Masked Somatic Mutation", 
                workflow.type = "Aliquot Ensemble Somatic Variant Merging and Masking")
GDCdownload(PRAD)
maf <- GDCprepare(PRAD)
saveRDS(maf, 'prad.maf')
```

## 清洗数据

```R
library(dplyr)
prad <- readRDS('prad.maf')
type <- as.numeric(substr(prad$Tumor_Sample_Barcode, 14, 15))
prad <- subset(prad, type < 10) # tp
group <- readRDS('../idea_2/fig3.2/fig5/tcga.predict.rds')
prad$BarCode <- substr(prad$Tumor_Sample_Barcode,1, 12)
group$BarCode  <- rownames(group)
prad <- subset(prad, prad$BarCode %in% group$BarCode)
prad <- left_join(x = prad, y = group, by = 'BarCode')
```

## 分两组导入maftools中

```R
library(maftools)
prad_l <- subset(prad, group=='Low Risk')
prad_h <- subset(prad, group=='High Risk')
maf_l <- read.maf(prad_l)
maf_h <- read.maf(prad_h)
```

## 比较并进行可视化

```R
lvsh <- mafCompare(m1=maf_l, m2=maf_h, m1Name="Low Risk", m2Name="High Risk", minMut=5)
saveRDS(lvsh, 'lvsh.rds')
```

### 森林图展示突变数量差异

```R
options(repr.plot.width=8, repr.plot.height=6)
forestPlot(mafCompareRes=lvsh, pVal=0.05, color=c("maroon", "royalblue"), geneFontSize=1.2)
```

### 瀑布图oncoplot展示突变景观

```R
options(repr.plot.width=12, repr.plot.height=8)
genes <- subset(lvsh$results, pval < 0.05)$Hugo_Symbol
coOncoplot(m1=maf_l, m2=maf_h, m1Name="Low Risk", m2Name="High Risk", genes=genes)
```

### 棒棒糖图深入特定基因突变细节

```R
options(repr.plot.width=12, repr.plot.height=6)
lollipopPlot2(m1=maf_l, m2=maf_h, m1_name="Low Risk", m2_name="High Risk", gene="TP53", AACol1 = "HGVSp_Short", AACol2 = "HGVSp_Short")
```