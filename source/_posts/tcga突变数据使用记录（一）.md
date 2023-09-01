---
title: TCGA突变数据使用记录（一）
tags: []
id: '2007'
categories:
  - - uncategorized
date: 2022-09-04 17:05:07
---

## 数据下载，从cBioPortal

*   从[cBioPortal](https://www.cbioportal.org/)找到相应的数据，并从[github](https://github.com/cBioPortal/datahub/tree/master/public)下载
*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://github.com/cBioPortal/datahub/raw/master/public/prad\_tcga/data\_mutations.txt
*   tmp <- read.table('data\_mutations.txt', header = T, sep = '\\t', allowEscapes = T, quote = '')

## 数据下载，从TCGA

*   访问[TCGA](https://portal.gdc.cancer.gov/repository)，在文件类型中过滤maf，访问权限中过滤open，case中过滤感兴趣的组织
*   加入到cart，然后可以直接下载tar.gz，也可以下载**manifest**，然后[使用](http://www.bio-info-trainee.com/2513.html)[官方工具](https://gdc.cancer.gov/access-data/gdc-data-transfer-tool)下载
*   后续可以使用Maftools进行分析