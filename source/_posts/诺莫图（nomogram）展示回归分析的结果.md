---
title: 诺莫图（Nomogram）展示回归分析的结果
tags: []
id: '1914'
categories:
  - - uncategorized
date: 2022-07-17 01:44:52
---

展现回归分析的OR、HR值可以用[森林图](https://occdn.limour.top/2097.html)，而如果想直观预测实例化对象的生存概率，则可以使用[诺莫图](https://zhuanlan.zhihu.com/p/84022664)

## 安装补充包

*   [conda activate ggsurvplot](https://occdn.limour.top/1820.html)
*   conda install -c conda-forge r-rms -y

## Logistic回归

*   [数据来源](https://occdn.limour.top/2099.html)

```R
dd=datadist(df)
options(datadist="dd") 
lrmf <- paste0("factor(dcf_status)~", paste(colnames(df)[1:10], collapse = '+'))
lrmf
f <- lrm(formula(lrmf) , data =  df)
nom <- nomogram(f, fun=plogis, lp=F, funlabel="Risk")
options(repr.plot.width=10, repr.plot.height=6)
plot(nom, xfrac=.2)
```

## **C**ox回归

```R
dd=datadist(df)
options(datadist="dd") 
coxmf <- paste0("Surv(dcf_time/365, dcf_status==0)~", paste(colnames(df)[1:10], collapse = '+'))
coxmf
f2 <- psm(formula(coxmf), data=df, dist='lognormal') # 构建COX比例风险模型
med <- Quantile(f2) # 计算中位生存时间
surv <- Survival(f2) # 构建生存概率函数
nom <- nomogram(f2, fun=function(x) med(lp=x),funlabel="Median Survival Time")
options(repr.plot.width=12, repr.plot.height=6)
plot(nom,xfrac=.2)
nom <- nomogram(f2, fun=list(function(x) surv(10, x)),
                             funlabel=c("10-year Survival Probability"))
options(repr.plot.width=10, repr.plot.height=6)
plot(nom, xfrac=.2)
```