---
title: Rstudio-server：安装seurat
tags: []
id: '1968'
categories:
  - - uncategorized
date: 2022-08-12 10:05:28
---

与[Jupyter](https://occdn.limour.top/2179.html)相比，使用[Rstudio](https://occdn.limour.top/1677.html)可以更方便进行交互式绘图，比如[ggThemeAssist](https://occdn.limour.top/1682.html)，以及某些3D绘图需要旋转视角进行观察。因此这里记录一下在Rstudio里安装seurat的过程

*   [更改R版本](https://occdn.limour.top/1680.html)
*   进入terminal，以下操作均在terminal中进行
*   export R\_LIBS\_SITE=""
*   在terminal中进入R
*   .libPaths('/home/rstudio/miniconda3/envs/r\_4\_1\_3/lib/R/library')
*   .libPaths() 确保没有其他路径
*   remove.packages("vctrs")
*   install.packages("vctrs")
*   install.packages('Seurat')
*   remove.packages("cli")
*   install.packages("cli")
*   install.packages("tidyverse")
*   重启R session
*   library(tidyverse)
*   library(Seurat)

*   如果conda有这个包，且环境不冲突，则简单一些。但这个Rstudio-server不支持conda虚拟环境。
*   进入terminal
*   conda install -c conda-forge r-seurat -y