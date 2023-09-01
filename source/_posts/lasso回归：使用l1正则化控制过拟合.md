---
title: LASSO回归：使用L1正则化控制过拟合
tags: []
id: '1959'
categories:
  - - uncategorized
date: 2022-08-10 00:35:52
---

## 安装补充包

*   [conda activate ggsurvplot](https://occdn.limour.top/2112.html)
*   conda install -c conda-forge r-glmnet -y

## LASSO回归

```R
library(glmnet)
library(survival)
dat <- readRDS('../../../COXPH/data/PRAD.rds')
y <- Surv(round(dat$meta$pfs_time,2), dat$meta$pfs_status == 1)
y <- data.matrix(y)
pfg <- readRDS('../../fig.2/D_ref_A_fiig.2_D/x.rds')
x <- dat$data[,pfg$RRA]
x <- as.matrix(x)
# x <- model.matrix(~.,clinical)
lasso <- glmnet(x = x, y = y, family = 'cox', alpha = 1)
```

## LASSO图确定λ与变量个数的关系

```R
lasso
pdf(file = 'lasso_p1.pdf');plot(lasso, xvar="lambda", label=T);dev.off()
lasso.coef <- predict(lasso, s = 0.039720, type = 'coefficients')
lasso.coef
```

## 交叉验证选择λ

```R
set.seed(0)
lasso.cv <- cv.glmnet(x,y,family="cox", alpha=1,nfolds=10)
pdf(file = 'lasso_cv.pdf');plot(lasso.cv);dev.off()
coef(lasso.cv, s="lambda.min")
coef(lasso.cv, s="lambda.1se")
lasso.cv$lambda.min
lasso.cv$lambda.1se
```

### 其他交叉验证评价指标

```R
set.seed(0)
lasso.cv_auc <- cv.glmnet(x,y,family="cox", alpha=1,nfolds=10, measure = 'auc')
plot(lasso.cv_auc)
```