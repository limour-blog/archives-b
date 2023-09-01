---
title: SingleR (一) 参考数据集的构建方法
tags: []
id: '656'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-09-20 00:36:50
---

## 第一步 下载所需的参考数据集

*   确定目标：[`Human Multiple Cortical Areas SMART-seq`](https://portal.brain-map.org/atlases-and-data/rnaseq/human-multiple-cortical-areas-smart-seq)
*   下载：[`Gene expression matrix`](https://idk-etl-prod-download-bucket.s3.amazonaws.com/aibs_human_ctx_smart-seq/matrix.csv) 和 [`Table of cell metadata`](https://idk-etl-prod-download-bucket.s3.amazonaws.com/aibs_human_ctx_smart-seq/metadata.csv)

```
mkdir Human_Multiple_Cortical_Areas_SMART-seq && cd Human_Multiple_Cortical_Areas_SMART-seq
wget https://idk-etl-prod-download-bucket.s3.amazonaws.com/aibs_human_ctx_smart-seq/matrix.csv
wget https://idk-etl-prod-download-bucket.s3.amazonaws.com/aibs_human_ctx_smart-seq/metadata.csv
```

*   同样方法下载：[`Human M1 10x`](https://portal.brain-map.org/atlases-and-data/rnaseq/human-m1-10x)

## 第二步 读入下载的参考数据集与注释

*   读入 [`Human M1 10x`](https://portal.brain-map.org/atlases-and-data/rnaseq/human-m1-10x)

```
ref_meta <- read.csv("/home/rqzhang/zlliu/R_data/human_M1_10x/metadata.csv")
ref_counts <- read.csv("/home/rqzhang/zlliu/R_data/human_M1_10x/matrix.csv")
```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-1.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-1.png)

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-3.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-3.png)

*   读入 [`Human Multiple Cortical Areas SMART-seq`](https://portal.brain-map.org/atlases-and-data/rnaseq/human-multiple-cortical-areas-smart-seq)

```
ref2_meta <- read.csv("/home/rqzhang/zlliu/R_data/Human_Multiple_Cortical_Areas_SMART-seq/metadata.csv")
ref2_counts <- read.csv("/home/rqzhang/zlliu/R_data/Human_Multiple_Cortical_Areas_SMART-seq/matrix.csv")
```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-4.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-4.png)

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-5.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-5.png)

## 第三步 转置矩阵

```
now_t <- Sys.time ()
ref_t_counts <- t(ref_counts) #超级慢，15分钟，似乎没有其他解决方案
Sys.time () - now_t

now_t <- Sys.time ()
ref2_t_counts <- t(ref2_counts) #超级慢，10分钟，似乎没有其他解决方案
Sys.time () - now_t

ref_d_counts <- data.frame(ref_t_counts[-1,], stringsAsFactors = F)
ref2_d_counts <- data.frame(ref2_t_counts[-1,], stringsAsFactors = F)

colnames(ref_d_counts) <- ref_t_counts[1,]
colnames(ref2_d_counts) <- ref2_t_counts[1,]

ref_d_counts <- as.data.frame(lapply(ref_d_counts, as.integer))
ref2_d_counts <- as.data.frame(lapply(ref2_d_counts, as.integer))

```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-7.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-7.png)

## 第四步 加载程辑包并计算SummarizedExperiment

```
library(SummarizedExperiment)
library(scater)

ref_d_counts <- SummarizedExperiment(assays=list(counts=ref_d_counts))
ref2_d_counts <- SummarizedExperiment(assays=list(counts=ref2_d_counts))

ref_d_counts <- logNormCounts(ref_d_counts)
ref2_d_counts <- logNormCounts(ref2_d_counts)

```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-8.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-8.png)

## 第五步 与metadata一起保存

```
ref_d_counts$meta <- ref_meta
ref2_d_counts$meta <- ref2_meta

```

[![](https://img-cdn.limour.top/blog_wp/2021/09/image-9.png)](https://img-cdn.limour.top/blog_wp/2021/09/image-9.png)