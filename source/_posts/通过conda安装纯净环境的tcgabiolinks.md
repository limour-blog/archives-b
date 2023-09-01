---
title: 通过conda安装纯净环境的TCGAbiolinks
tags: []
id: '1637'
categories:
  - - 超算基础
date: 2022-02-15 20:53:34
---

*   conda create -n tcga -c conda-forge r-base=4.1.2
*   conda activate tcga
*   conda install -c conda-forge r-rvest=1.0.2
*   conda install -c conda-forge r-xml=3.99\_0.8
*   conda install -c conda-forge r-rcpparmadillo=0.10.8.1.0
*   conda install -c conda-forge r-bh=1.78.0\_0
*   R
*   install.packages("BiocManager")
*   BiocManager::install("SummarizedExperiment")
*   **选择更新rlang**
*   BiocManager::install("TCGAbiolinks")
*   BiocManager::install("DESeq2")
*   q()
*   conda install -c conda-forge r-irkernel=1.3

永远可以对conda-forge的包抱有希望，永远不要对bioconda的包抱有希望