---
title: TCGA (一) 获取数据
tags: []
id: '770'
categories:
  - - 生物信息学
  - - 组织测序分析
date: 2021-09-24 00:28:13
---

## 第一步 安装程辑包并加载

```
options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
BiocManager::install("TCGAbiolinks")

# 加载响应的包，默认已经安装好TCGAbiolinks包
library(TCGAbiolinks)
library(plyr)
library(limma)
library(biomaRt)
library(SummarizedExperiment)
```

## 第二步 查看癌症类型

```
TCGAbiolinks:::getGDCprojects()$project_id
```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-23.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-23.png)

癌症类型说明：[癌症类型和样本代号详解TCGA](https://www.biowolf.cn/TCGA/tcga_sample.html)

## 第三步 查看对应癌症的数据类型

```
TCGAbiolinks:::getProjectSummary('TCGA-PRAD') # 以前列腺癌为例
```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-24.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-24.png)

case\_count为病人数，file\_count为对应的文件数，"Transcriptome Profiling"表示表达谱

> [“如何玩转生物大数据”系列：TCGA的样本注释信息和数据类型统计](http://blog.sciencenet.cn/blog-3291578-1066017.html#:~:text=%E5%9C%A8TCGA%E4%B8%AD%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%9C%89%E4%B8%8B%E9%9D%A2%E7%9A%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%EF%BC%9A%201%EF%BC%89%E8%BD%AC%E5%BD%95%E7%BB%84%E6%95%B0%E6%8D%AE%EF%BC%88Transcriptome%EF%BC%89,2%EF%BC%89%E7%94%B2%E5%9F%BA%E5%8C%96%E6%95%B0%E6%8D%AE%EF%BC%88Methylation%EF%BC%89%203%EF%BC%89%E5%9F%BA%E5%9B%A0%E7%AA%81%E5%8F%98%E6%95%B0%E6%8D%AE%EF%BC%88Mutation%EF%BC%89)
> 
> 1）转录组数据（Transcriptome）  
> 2）甲基化数据（Methylation）  
> 3）基因突变数据（Mutation）  
> 4）拷贝数变化数据 （CNV）

> [手把手教你 TCGA 数据库使用：以肝癌为例](http://paper.dxy.cn/article/511878)
> 
> ![wxid_7vqx3f6qn62f12_1482131772510_79.png](http://img.dxycdn.com/cms/upload/userfiles/image/2016/12/19/B1482129119_small.jpg)  
> ![wxid_7vqx3f6qn62f12_1482131772670_15.png](http://img.dxycdn.com/cms/upload/userfiles/image/2016/12/19/B1482129120_small.jpg)

> [TCGAbiolinks包下载TCGA数据进行表达差异分析-乳腺癌案例](https://cloud.tencent.com/developer/article/1481904)
> 
> （4）workflow.type  
> 该数据类型有很多种，根据data.type的不同而不同，不同的数据类型，有其对应的参数可供选择。比如Gene Expression Quantification数据类型下workflow.type 有4种类型分别为：  
> HTSeq - FPKM-UQ：FPKM上四分位数标准化值  
> HTSeq - FPKM：FPKM值/表达量值  
> HTSeq - Counts：原始count数  
> STAR - Counts  
> 具体可在GDC官网查看  
> （5）legacy  
> 这个参数主要是因为TCGA数据有两个入口可以下载，GDC Legacy Archive 和 GDC Data Portal，区别主要是注释参考基因组版本不同分别是：GDC Legacy Archive（hg19和GDC Data Portal（hg38）。参数默认为FALSE，下载GDC Data Portal（hg38）。这里建议是，下载转录组层面的数据使用hg38，下载DNA层面的数据使用hg19，因为比如做SNP分析的时候很多数据库没有hg38版本的数据，都是hg19的。

## 第四步 下载对应数据

```
query <- GDCquery(project = 'TCGA-PRAD',
                  data.category = "Transcriptome Profiling",
                  data.type = "Gene Expression Quantification", 
                  workflow.type = "HTSeq - Counts")
```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-25.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-25.png)

## 第五步 保存对应数据

```
# 配置数据路径
root_path = "~/zlliu/R_data/TCGA"
 
# 配置结果保存路径
output_path = root_path
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

GDCdownload(query = query)
saveRDS(query,'TCGA-PRAD.rds')
```