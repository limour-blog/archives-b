---
title: cBioPortal的原始数据获取
tags: []
id: '1680'
categories:
  - - 组织测序分析
date: 2022-02-18 11:02:21
---

cBioPortal中的文章相关数据都存放在这里：[https://github.com/cBioPortal/datahub/tree/master/public](https://github.com/cBioPortal/datahub/tree/master/public)

## 第一步 获取需要的数据

```shell
wget https://github.com/cBioPortal/datahub/raw/master/public/prad_su2c_2019/data_mrna_seq_fpkm_capture.txt
```

## 第二步 读取数据

```r
f_name_dedup <- function(lc_exp, rowN = 1){
    res <- lc_exp[-rowN]
    lc_tmp = by(res,
         lc_exp[[rowN]],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res = lc_exp[rownames(res) %in% lc_probes,]
    rownames(res) <- res[[rowN]]
    res[-rowN]
}
```

```r
d <- read.table('data_mrna_seq_fpkm_capture.txt', header = T, sep = '\t', allowEscapes = T)
d <- f_name_dedup(d)
d <- d[,order(colnames(d))]
```

## 第三步 导出需要的基因

```r
write.csv(t(log1p(d[c('RPLP0', 'GAPDH'),])), file='prad_su2c_2019.csv.csv')
```