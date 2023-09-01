---
title: TCGA (五) 反卷积推断细胞构成比
tags: []
id: '1060'
categories:
  - - 生物信息学
  - - 组织测序分析
date: 2021-10-15 01:48:53
---

[SCDC: Bulk Gene Expression Deconvolution by Multiple Single-Cell RNA Sequencing References](https://github.com/meichendong/SCDC)

## 第一步 安装依赖

*   install.packages("remotes")
*   remotes::install\_github("renozao/xbioc")
*   proxychains4 wget https://github.com/meichendong/SCDC/archive/refs/heads/master.zip
*   devtools::install\_local("master.zip")