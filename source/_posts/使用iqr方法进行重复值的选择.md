---
title: 使用IQR方法进行重复值的选择
tags: []
id: '1933'
categories:
  - - uncategorized
date: 2022-07-27 08:22:17
---

使用 `rownames(df) <- rowNn` 时经常会遇到`rowNn`中有重复值的情况，此时需要使用合适的策略来选择需要保留的那一列。下面这个函数默认保留IQR值(四分位距)最大的那一列。通过传入不同的`select_func`参数值，也可以改用其他的保留选择策略。如 `mean` 来保留算数平均值最大的一列，也可以传入自己定义的函数。

来源：[Comprehensive Evaluation of Machine Learning Models and Gene Expression Signatures for Prostate Cancer Prognosis Using Large Population Cohorts](https://doi.org/10.1158/0008-5472.CAN-21-3074)

```R
f_rm_duplicated <- function(NameL, reverse=F){
    tmp <- data.frame(table(NameL))
    if(reverse){
        tmp <- tmp$NameL[tmp$Freq > 1]
    }else{
        tmp <- tmp$NameL[tmp$Freq == 1]
    }
    which(NameL %in% as.character(tmp))
}
f_dedup_IQR <- function(df, rowNn, select_func='IQR'){
    if(typeof(select_func) == 'character'){
        select_func = get(select_func)
    }
    # 拆出无重复的数据，后续不进行处理
    noDup <- f_rm_duplicated(rowNn)
    tmp <- rowNn[noDup]
    noDup <- df[noDup,]
    rownames(noDup) <- tmp
    # 拆除有重复的数据
    Dup <- f_rm_duplicated(rowNn, T)
    rowNn <- rowNn[Dup]
    Dup <- df[Dup,]
    rownames(Dup) <- NULL
    # 处理重复的数据
    lc_tmp = by(Dup,
         rowNn,
         function(x){rownames(x)[which.max(apply(X = x, FUN = select_func, MARGIN = 1))]})
    lc_probes = as.integer(lc_tmp)
    Dup = Dup[lc_probes,]
    rownames(Dup) <- rowNn[lc_probes]
    # 合并数据并返回
    return(rbind(noDup,Dup))
}
```