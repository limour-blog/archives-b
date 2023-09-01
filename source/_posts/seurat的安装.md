---
title: Seurat (一) 安装
tags: []
id: '643'
categories:
  - - Seurat教程
  - - 生物信息学
  - - 超算基础
date: 2021-09-19 21:40:07
---

*   安装BiocManager：`install.packages("BiocManager", repos="http://mirrors.tuna.tsinghua.edu.cn/CRAN/" )`
*   为BiocManager添加镜像：`options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")`
*   安装Seurat：`BiocManager::install("Seurat")`
*   安装devtools：`BiocManager::install("devtools")`
*   通过devtools在线安装：`devtools::install_github("github_user_name/package_name")`
*   通过devtools离线安装：`devtools::install_local("path_to_package_file.zip")`
*   查看或设置lib路径：`.libPaths("<dirPath>")`
*   离线安装包：`install.packages("<filePath>", repos = NULL)`
*   命令行安装离线包：`R CMD INSTALL package.tar.gz`