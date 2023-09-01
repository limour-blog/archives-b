---
title: 通过conda安装纯净环境的WGCNA
tags: []
id: '1906'
categories:
  - - uncategorized
date: 2022-07-15 00:16:18
---

*   conda create -n wgcna -c conda-forge r-base=4.1.3
*   conda activate wgcna
*   conda install -c conda-forge r-tidyverse=1.3.1 -y
*   conda install -c conda-forge r-gplots -y
*   conda install -c bioconda r-wgcna -y
*   conda install -c conda-forge r-irkernel -y
*   Rscript -e "IRkernel::installspec(name='wgcna', displayname='r-wgcna')"