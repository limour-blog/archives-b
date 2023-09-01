---
title: TCGAbiolinks下载CNV数据
tags: []
id: '2009'
categories:
  - - uncategorized
date: 2022-09-07 16:55:29
---

## 下载Gene水平的数据

```R
library(TCGAbiolinks)
query <- GDCquery(
    project = "TCGA-PRAD",
    data.category = "Copy Number Variation",
    data.type = "Gene Level Copy Number",              
    access = "open"
)
GDCdownload(query)
data <- GDCprepare(query)
saveRDS(data, 'prad_cnv.rds')
```

## 下载Masked数据

```R
query <- GDCquery(
    project = "TCGA-PRAD",
    data.category = "Copy Number Variation",
    data.type = "Masked Copy Number Segment",              
    access = "open"
)
GDCdownload(query)
data <- GDCprepare(query)
saveRDS(data, 'prad_cnv_masked.rds')
```

## 清洗数据

### 初步清洗

```R
library(SummarizedExperiment)
data <- readRDS('prad_cnv.rds')
cnT <- data@assays@data$copy_number
cnTcol <- colnames(cnT)
type <- as.numeric(substr(cnTcol, 14, 15))
cnT <- cnT[, type<10]
colnames(cnT) <- substr(cnTcol,1, 12)
rownames(cnT) <-  data@rowRanges$gene_name
cnT <- na.omit(cnT)
```

### 精细清洗

[f\_dedup\_IQR](https://occdn.limour.top/2157.html)

```R
cnT <- f_dedup_IQR(cnT, rownames(cnT))
cnT <- cnT[,f_rm_duplicated(colnames(cnT))]
group <- readRDS('../idea_2/fig3.2/fig5/tcga.predict.rds')
cnT <- cnT[,colnames(cnT) %in% rownames(group)]
```

### 构造cnTable

#### 慢速

```R
df <- NULL
for (i in 1:ncol(cnT)){
    colnames(cnT)[[i]]
    tmp_df <- data.frame(Hugo_Symbol = rownames(cnT), Tumor_Sample_Barcode = colnames(cnT)[[i]], Variant_Classification=cnT[,i])
    tmp_df <- subset(tmp_df, Variant_Classification != 2)
    df <- rbind(df, tmp_df)
}
```

#### 快速

```R
library(reshape2)
df <- melt(cnT)
colnames(df) = c('Hugo_Symbol', 'Tumor_Sample_Barcode', 'Variant_Classification')
df <- subset(df, Variant_Classification != 2)
df
```

#### 贴标签

```R
df$Variant_Classification[df$Variant_Classification > 2]  <- 'Amp'
df$Variant_Classification[df$Variant_Classification < 2]  <- 'Del'
table(df$Variant_Classification)
```

### 分组别

```R
df_l <- subset(df, Tumor_Sample_Barcode %in% rownames(group)[group$group == 'Low Risk'])
df_h <- subset(df, Tumor_Sample_Barcode %in% rownames(group)[group$group == 'High Risk'])
saveRDS(df_l, 'cnT.l.rds')
saveRDS(df_h, 'cnT.h.rds')
```

## 导入maftools

[TCGAbiolinks下载maf数据](https://occdn.limour.top/2304.html)

### 清洗数据

```R
cnv_l <- readRDS('cnT.l.rds')
cnv_h <- readRDS('cnT.h.rds')
prad_l$Tumor_Sample_Barcode <- prad_l$BarCode
prad_l <- subset(prad_l, Tumor_Sample_Barcode %in% cnv_l$Tumor_Sample_Barcode)
cnv_l <- subset(cnv_l, Tumor_Sample_Barcode %in% prad_l$Tumor_Sample_Barcode)
prad_h$Tumor_Sample_Barcode <- prad_h$BarCode
prad_h <- subset(prad_h, Tumor_Sample_Barcode %in% cnv_h$Tumor_Sample_Barcode)
cnv_h <- subset(cnv_h, Tumor_Sample_Barcode %in% prad_h$Tumor_Sample_Barcode)
```

### 读入maftools

```R
maf_l <- read.maf(prad_l, cnTable = cnv_l)
maf_h <- read.maf(prad_h, cnTable = cnv_h)
```

### 绘制瀑布图

```R
options(repr.plot.width=12, repr.plot.height=8)
genes <- subset(lvsh$results, pval < 0.05)$Hugo_Symbol
coOncoplot(m1=maf_l, m2=maf_h, m1Name="Low Risk", m2Name="High Risk",genes=genes)
```