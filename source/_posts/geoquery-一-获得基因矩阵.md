---
title: GEOquery (一) 获得基因矩阵
tags: []
id: '1677'
categories:
  - - 组织测序分析
date: 2022-02-18 09:35:05
---

## 第一步 定义注释函数

```r
f_getGPL <- function(lc_GPLN, lc_local = F){
    options(stringsAsFactors = F)
    if (!file.exists(lc_GPLN)){dir.create(lc_GPLN)}
    if(lc_local){
        gpl=read.table(file.path(lc_GPLN, lc_GPLN),
               header = TRUE,fill = T,sep = "\t",
               comment.char = "#",
               stringsAsFactors = FALSE,
               quote = "")
        return(gpl)
    }else{
        return(Table(getGEO(lc_GPLN,destdir = lc_GPLN)))
    }
}
f_id2name <- function(lc_exp, lc_db){
    if(!is.data.frame(lc_db)){
        lc_ids <- toTable(lc_db)
    }else{
        lc_ids <- lc_db
    }
    res <- lc_exp[rownames(lc_exp) %in% lc_ids[[1]],]
    lc_ids=lc_ids[match(rownames(res),lc_ids[[1]]),]
    lc_tmp = by(res,
         lc_ids[[2]],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res = res[rownames(res) %in% lc_probes,]
    rownames(res)=lc_ids[match(rownames(res),lc_ids[[1]]),2]
    res
}
```

## 第二步 获得基础数据

*   在线方式

```r
library(GEOquery)
gset <- getGEO("GSE41192", GSEMatrix =TRUE, getGPL=FALSE)
gset <- gset[[1]] # 自行选择合适的值
GPL <- f_getGPL( annotation(gset))
GPL <- GPL[c('ID', 'GENE_SYMBOL')] # 自行选择合适的值
d <- exprs(gset) # 自行判断是否需要log1p
colnames(d) <- gset$title
d <- f_id2name(d, GPL)
```

*   离线方式

```r
library(GEOquery)
d <- read.table('GSE130247_22RV1_BETi_RNASeq.FPKM.txt', header = T, row.names = 1, sep = '\t', allowEscapes = T)
tt <- getGEO(filename = 'GSE130247_series_matrix.txt.gz')
tt <- t(as.data.frame((strsplit(tt$title, ': '))))
colnames(d) <- tolower(colnames(d))
d <- d[, tt[,1]]
colnames(d)  <- tt[,2]
```

## 第三步 导出需要的基因

```r
write.csv(t(log1p(d[c('RPLP0', 'GAPDH'),])), file='GSE41192.csv')
```