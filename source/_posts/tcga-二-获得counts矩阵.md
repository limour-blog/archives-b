---
title: TCGA (二) 获得counts矩阵
tags: []
id: '779'
categories:
  - - 生物信息学
  - - 组织测序分析
date: 2021-09-25 13:27:02
---

## 第一步 进入对应的[癌症页面](https://portal.gdc.cancer.gov/)

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-26.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-26.png)

## 第二步 查看所需的Project

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-27.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-27.png)

## 第三步 下载所需的分组

```
# 一般的前列腺癌 GDC Data Portal 是 hg38 的
PRAD <- GDCquery(project = 'TCGA-PRAD',
                  data.category = "Transcriptome Profiling",
                  data.type = "Gene Expression Quantification", 
                  workflow.type = "HTSeq - Counts")
# mCRPC
mCRPC <- GDCquery(project = 'WCDT-MCRPC',
              data.category = "Transcriptome Profiling",
              data.type = "Gene Expression Quantification", 
              workflow.type = "HTSeq - Counts")

# 选择病例列 ，不加cols参数则是完整结果的全部列
PRAD_cases <- getResults(PRAD,cols=c("cases"))
mCRPC_cases <- getResults(mCRPC,cols=c("cases"))

# 选择癌组织数据
PRAD_tp  <- TCGAquery_SampleTypes(barcode = PRAD_cases, typesample = "TP")

PRAD_D <- GDCquery(project = 'TCGA-PRAD',
                  data.category = "Transcriptome Profiling",
                  data.type = "Gene Expression Quantification", 
                  workflow.type = "HTSeq - Counts",
                  barcode = PRAD_tp)
mCRPC_D <- GDCquery(project = 'WCDT-MCRPC',
              data.category = "Transcriptome Profiling",
              data.type = "Gene Expression Quantification", 
              workflow.type = "HTSeq - Counts",
              sample.type = "Metastatic", #下载癌组织的数据
              barcode = mCRPC_cases)
saveRDS(PRAD_D,"PRAD_D.rds")
saveRDS(mCRPC_D,"mCRPC_D.rds")

GDCdownload(query = PRAD_D)
GDCdownload(query = mCRPC_D)

PRAD_pre1 <- GDCprepare(query = PRAD_D, save = TRUE, save.filename = "PRAD_pre1.rda")
mCRPC_pre1 <- GDCprepare(query = mCRPC_D, save = TRUE, save.filename = "mCRPC_pre1.rda")

# 导出counts矩阵
PRAD_pre <- PRAD_pre1@assays@data$`HTSeq - Counts`
colnames(PRAD_pre) <- PRAD_pre1@colData@rownames
rownames(PRAD_pre) <- PRAD_pre1@rowRanges$external_gene_name

mCRPC_pre <- mCRPC_pre1@assays@data$`HTSeq - Counts`
colnames(mCRPC_pre) <- mCRPC_pre1@colData@rownames
rownames(mCRPC_pre) <- mCRPC_pre1@rowRanges$external_gene_name

```