---
title: 通过conda安装纯净环境的Seurat
tags: []
id: '1619'
categories:
  - - 超算基础
date: 2022-02-13 16:05:35
---

*   conda create -n seurat -c conda-forge r-base=4.1.2
*   conda activate seurat
*   conda install -c conda-forge r-seurat=4.1.0
*   conda install -c conda-forge r-irkernel
*   R
*   IRkernel::installspec(name='seurat', displayname='r-seurat')
*   q()

经过长期实践，用好conda真能省心高效方便！建议不要在base环境中安装除conda之外的任何包（有些包会和conda的依赖冲突，此时用conda创建的新环境就可以避免这个问题），然后每一个独立的功能都单独建一个新环境。

注：如果出现失败，一般是默认安装的版本太低，此时应使用r-base=4.1.2来指定最新版本。可以在[此](https://anaconda.org/search?q=r-seurat)搜索最新的版本：[https://anaconda.org/search?q=r-seurat](https://anaconda.org/search?q=r-seurat)

**重要的事情说三遍：不要自行尝试编译安装，维护起来很难！不要自行尝试编译安装，维护起来很难！不要自行尝试编译安装，维护起来很难！**