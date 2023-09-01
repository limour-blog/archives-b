---
title: Seurat (七) 判断某个细胞群的性质 (二) ——熵分析预测细胞干性
tags: []
id: '1055'
categories:
  - - Seurat教程
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-10-15 01:46:39
---

[SLICE：Determining Cell Differentiation and Lineage based on Single Cell Entropy](https://www.jianshu.com/p/519071290ecf)

## [SLICE: Determining Cell Differentiation and Lineage based on Single Cell Entropy](https://research.cchmc.org/pbge/slice.html)

## 第一步 安装依赖

*   BiocManager::install(c('Biobase', 'entropy', 'graph', 'BioNet', 'cluster', 'princurve', 'lmtest', 'mgcv', 'gridExtra'))
*   proxychains4 wget https://research.cchmc.org/pbge/slice/downloads/SLICE.a12242016.zip
*   unzip SLICE.a12242016.zip