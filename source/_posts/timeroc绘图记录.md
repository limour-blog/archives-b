---
title: timeROC绘图记录
tags: []
id: '1988'
categories:
  - - uncategorized
date: 2022-08-19 05:55:03
---

## 多变量

```R
library(survival)
library(car)
library(rms)
library(pROC)
library(timeROC)
library(ggDCA)
library(survivalROC)
library(pec)
require(ggsci)
library(scales)
library(nomogramFormula)
library(survIDINRI)
dat <- readRDS('dat.rds')
f_timeROC <-function(dat, geneNs){
    coxmf2 <- paste0("Surv(time, status==1)~", paste(paste('`',geneNs,'`',sep = ''), collapse = '+'))
    f2 <- cph(formula(coxmf2), data=dat, x=T, y=T, surv = T) # 构建COX比例风险模型
    years2 <- timeROC(T=dat$time,delta=dat$status,
                          marker=predict(f2,type="lp",newdata=dat),cause=1,
                          weighting="marginal",
        #                   other_markers=as.matrix(df$`1-year`),
                          times=12*c(1,3,5), ROC=T, iid=T)
    auc2.ci <- confint(years2, parm=NULL, level = 0.95,n.sim=2000)
    auc2.ci <- auc2.ci$CI_AUC
    auc2.ci
    years2.auc1 <- paste0(round(years2$AUC[1],4)*100,'(',auc2.ci[1,1],'~',auc2.ci[1,2],')%')
    years2.auc2 <- paste0(round(years2$AUC[2],4)*100,'(',auc2.ci[2,1],'~',auc2.ci[2,2],')%')
    years2.auc3 <- paste0(round(years2$AUC[3],4)*100,'(',auc2.ci[3,1],'~',auc2.ci[3,2],')%')
    res <- list(roc=years2, auc1=years2.auc1, auc2=years2.auc2, auc3=years2.auc3)
    res
}
options(repr.plot.width=8, repr.plot.height=8)
tmp <- f_timeROC(dat, 'Risk_Score')
plot(tmp$roc,time=12*1,title="Time dependent ROC",col="#BC3C29FF")        
plot(tmp$roc,time=12*3,add=T,col="#0072B5FF") 
plot(tmp$roc,time=12*5,add=T,col="#E18727FF") 
legend("bottomright",c(paste("AUC of 1-year =",tmp$auc1),
                 paste("AUC of 3-year =",tmp$auc2),
                 paste("AUC of 5-year =",tmp$auc3)),
       col=c("#BC3C29FF","#0072B5FF","#E18727FF"),lty=1,lwd=2)
```

## 单变量

```R
f_timeROC_one <-function(dat, geneN){
    years2 <- timeROC(T=dat$time,delta=dat$status,
                          marker=dat[[geneN]],cause=1,
                          weighting="marginal",
        #                   other_markers=as.matrix(df$`1-year`),
                          times=12*c(1,3,5), ROC=T, iid=T)
    auc2.ci <- confint(years2, parm=NULL, level = 0.95,n.sim=2000)
    auc2.ci <- auc2.ci$CI_AUC
    auc2.ci
    years2.auc1 <- paste0(round(years2$AUC[1],4)*100,'(',auc2.ci[1,1],'~',auc2.ci[1,2],')%')
    years2.auc2 <- paste0(round(years2$AUC[2],4)*100,'(',auc2.ci[2,1],'~',auc2.ci[2,2],')%')
    years2.auc3 <- paste0(round(years2$AUC[3],4)*100,'(',auc2.ci[3,1],'~',auc2.ci[3,2],')%')
    res <- list(roc=years2, auc1=years2.auc1, auc2=years2.auc2, auc3=years2.auc3)
    res
}
dat$PATH_T_STAGE <- factor(dat$PATH_T_STAGE, ordered = T)
dat$PATH_T_STAGE <- as.numeric(dat$PATH_T_STAGE)
options(repr.plot.width=8, repr.plot.height=8)
tmp <- f_timeROC_one(dat, 'PATH_T_STAGE')
plot(tmp$roc,time=12*1,title="Time dependent ROC",col="#BC3C29FF")        
plot(tmp$roc,time=12*3,add=T,col="#0072B5FF") 
plot(tmp$roc,time=12*5,add=T,col="#E18727FF") 
legend("bottomright",c(paste("AUC of 1-year =",tmp$auc1),
                 paste("AUC of 3-year =",tmp$auc2),
                 paste("AUC of 5-year =",tmp$auc3)),
       col=c("#BC3C29FF","#0072B5FF","#E18727FF"),lty=1,lwd=2)
```