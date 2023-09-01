---
title: 使用替换系数法使coxnet-lasso兼容coxph的分析
tags: []
id: '1965'
categories:
  - - uncategorized
date: 2022-08-11 00:58:48
---

目前R中用于构建LASSO模型的包主要是glmnet，当我们确定λ后，该如何画列线图、森林图等常规COXPH回归常用的可视化方法呢？[Memo\_Cleon](https://zhuanlan.zhihu.com/p/535759132)指出可以提取lasso的回归系数然后替换常规回归模型的系数。

## 导出lasso系数

```R
lasso <- glmnet(x = x, y = y, family = 'cox', alpha = 1)
set.seed(0)
lasso.cv <- cv.glmnet(x,y,family="cox", alpha=1,nfolds=10, type.measure = 'C')
lasso.cf <- coef(lasso.cv, s="lambda.1se")
lasso.cf <- as.data.frame(as.matrix(lasso.cf))
names(lasso.cf) <- 'coef'
saveRDS(lasso.cf, 'lasso.cf.rds')
```

## 替换rms包的cph结果的系数

```R
lasso.cf <- readRDS('../A_ref_A_fiig.3_A_B/lasso.cf.rds')
f1$coefficients <- lasso.cf[names(f1$coefficients),'coef']
```

## 替换survival包的coxph结果的系数

```R
res.cox$coefficients <- lasso.cf[names(res.cox$coefficients),'coef']
```

注意：coxph系数替换后其他量不会重新计算，基本没用