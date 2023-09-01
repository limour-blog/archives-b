---
title: 通过conda安装纯净环境的SingleR
tags: []
id: '2042'
categories:
  - - uncategorized
date: 2022-10-03 12:43:51
---

*   conda create -n singleR -c conda-forge r-base=4.1.2 -y
*   conda activate singleR
*   conda install -c conda-forge r-seurat=4.1.0 -y
*   conda install -c conda-forge r-irkernel=1.3 -y
*   conda install -c conda-forge r-biocmanager=1.30.16 -y
*   conda install -c conda-forge r-viridis=0.6.2 -y
*   conda install -c conda-forge r-pheatmap=1.0.12 -y
*   BiocManager::install("SingleR")
*   BiocManager::install("celldex")
*   IRkernel::installspec(name='singleR', displayname='r-singleR')