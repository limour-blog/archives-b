---
title: 通过conda安装纯净环境的GEOquery
tags: []
id: '1653'
categories:
  - - 超算基础
date: 2022-02-16 11:55:38
---

*   conda create -n geo -c conda-forge r-base=4.1.2
*   conda activate geo
*   conda install -c conda-forge r-irkernel=1.3
*   conda install -c conda-forge r-biocmanager=1.30.16
*   R
*   BiocManager::install("GEOquery")