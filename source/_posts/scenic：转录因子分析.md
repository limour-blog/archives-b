---
title: 通过conda安装纯净环境的SCENIC
tags: []
id: '1572'
categories:
  - - 超算基础
date: 2022-02-12 02:35:25
---

*   https://scenic.aertslab.org/tutorials/
*   https://anaconda.org/conda-forge/r-base
*   conda create -n scenic -c conda-forge r-reticulate
*   conda activate scenic
*   install.packages("BiocManager")
*   BiocManager::install("AUCell")
*   conda install -c conda-forge --strict-channel-priority r-arrow
*   conda install -c bioconda bioconductor-rcistarget
*   BiocManager::install(c("GENIE3"))
*   BiocManager::install(c("zoo", "mixtools", "rbokeh"))
*   BiocManager::install(c("DT", "NMF", "ComplexHeatmap", "R2HTML", "Rtsne"))
*   BiocManager::install(c("doMC", "doRNG"))
*   install.packages("devtools")
*   devtools::install\_local("SCopeLoomR-master.zip")
*   devtools::install\_local("SCENIC-master.zip")
*   library(SCENIC)
*   https://limour.top/1098.html
*   install.packages(c('repr', 'IRdisplay', 'evaluate', 'crayon', 'pbdZMQ', 'uuid', 'digest', 'Cairo', 'IRkernel'), repos="http://mirrors.tuna.tsinghua.edu.cn/CRAN/" )
*   IRkernel::installspec(name='scenic', displayname='r-scenic')