---
title: 通过conda安装纯净环境的cellchat
tags: []
id: '1632'
categories:
  - - 超算基础
date: 2022-02-13 20:05:39
---

*   先安装好纯净环境的Seurat：[https://limour.top/1619.html](https://limour.top/1619.html)
*   jupyter内核一般安装在：~/.local/share/jupyter/kernels/XXX  
    XXX文件夹内容示例：[https://limour.lanzouo.com/iv1PP000ysyh](https://limour.lanzouo.com/iv1PP000ysyh)
*   conda install -c conda-forge r-circlize=0.4.14
*   conda install -c conda-forge r-devtools
*   conda install -c bioconda bioconductor-biobase=2.54.0
*   wget https://github.com/sqjin/CellChat/archive/refs/heads/master.zip -O CellChat-master.zip
*   wget https://github.com/jokergoo/ComplexHeatmap/archive/refs/heads/master.zip -O ComplexHeatmap-master.zip
*   **移除CellChat-master.zip里src目录下的\*.so和\*.o文件并重新打包**
*   R （注意不要进错环境，可以用绝对路径）
*   install.packages('NMF')
*   devtools::install\_local('ComplexHeatmap-master.zip')
*   devtools::install\_local('CellChat-master.zip')

[![](https://img.limour.top/archives_2023/blog_wp/2022/02/image-3.webp)](https://img.limour.top/archives_2023/blog_wp/2022/02/image-3.webp)