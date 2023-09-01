---
title: 通过conda安装纯净环境的inferCNV
tags: []
id: '2041'
categories:
  - - uncategorized
date: 2022-10-03 12:28:08
---

*   conda create -n rinferCNV -c conda-forge r-base=4.1.2 -y
*   conda activate rinferCNV
*   conda install -c conda-forge r-seurat=4.1.0 -y
*   conda install -c conda-forge r-irkernel=1.3 -y
*   conda install -c conda-forge jags=4.3.0 -y
*   conda install -c conda-forge r-rjags=4\_12 -y
*   conda install -c conda-forge python=3.9.10 -y
*   conda install -c conda-forge r-biocmanager=1.30.16 -y
*   BiocManager::install("infercnv")
*   IRkernel::installspec(name='rinferCNV', displayname='r-inferCNV')
*   https://github.com/harbourlab/uphyloplot2
*   unzip uphyloplot2-master.zip