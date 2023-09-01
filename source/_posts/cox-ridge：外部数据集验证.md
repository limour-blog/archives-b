---
title: COX-Ridge：外部数据集验证
tags: []
id: '1990'
categories:
  - - uncategorized
date: 2022-08-19 08:42:29
---

## 建模

```R
library(glmnet)
library(survival)
df1 <- readRDS('../../fig.3/B_ref_A_fiig.3_C/new_df1.rds')
y <- data.matrix(df1[9:10])
x <- data.matrix(df1[1:8])
Ridge <- glmnet(x = x, y = y, family = 'cox', alpha = 0)
plot(Ridge, xvar = 'lambda', label = T)
set.seed(0)
Ridge.cv <- cv.glmnet(x,y,family="cox", alpha=0,nfolds=10, type.measure = 'C')
plot(Ridge.cv)
Ridge.cf <- coef(Ridge.cv, s="lambda.min")
Ridge.cf <- as.data.frame(as.matrix(Ridge.cf))
names(Ridge.cf) <- 'coef'
saveRDS(Ridge.cf, 'Ridge.cf.rds')
```

## 验证

```R
library("survival")
library("survminer")
library("rms")
train_sets <- readRDS('../../../COXPH/train_sets.rds')
test_sets <- readRDS('../../../COXPH/test/test_sets.rds')
all_sets <- c(train_sets, test_sets)
library(survival)
library(survminer)
library(ggpubr)
require(survRM2)
f_surv <- function(df, isplot=T, timeN='time', statusN='status', groupN='group', tau=5, risk.table=T, ncensor.plot=T){
    res <- list()
    my.surv <<- Surv(df[[timeN]], df[[statusN]]==1)
    my.surv.f <<- paste0("my.surv~", groupN)
    my.surv.df <<- df
    kmfit <- surv_fit(formula(my.surv.f), data = my.surv.df)
    sdiff <- survdiff(formula(my.surv.f), data = my.surv.df)
    res$pv <- 1 - pchisq(sdiff$chisq, length(sdiff$n) - 1)
    if(isplot){
        res$plot <- ggsurvplot(kmfit, conf.int =F, pval = T, risk.table = risk.table, ncensor.plot = ncensor.plot)
    }
    res$RMS <- rmst2(df[[timeN]], df[[statusN]], as.numeric(df[[groupN]])-1, tau=tau)
    res$RMST <- as.data.frame(rbind(res$RMS$RMST.arm1$result[1,],res$RMS$RMST.arm0$result[1,]))
    rownames(res$RMST) <- c('RMST (arm=1)', 'RMST (arm=0)')
    res
}
f_COXPH_predict <- function(data_sets, coxM){
    res <- list()
    for(Name in names(data_sets)){
        try(expr = {
            dat <- data_sets[[Name]]
            y <- Surv(round(dat$meta$pfs_time,2), dat$meta$pfs_status == 1)
            y <- data.matrix(y)
            x <- predict(f1,type="lp",newdata=dat$data)
            df <- as.data.frame(cbind(x,y))
            names(df)[1] <- 'Risk_Score'
            df$group <- 'Low Risk'
            df$group[df$Risk_Score > median(df$Risk_Score)] <- 'High Risk'
            df$group <- as.factor(df$group)
            res[[Name]] <- df
        })
    }
    res
} 
f_surv_list  <- function(df_lists, ...){
    res <- list()
    for(Name in names(df_lists)){
        df <- df_lists[[Name]]
        res[[Name]] <- f_surv(df, ...)
    }
    res
}
f1 <- readRDS('../../fig.3/B_ref_A_fiig.3_C/coxph.rds')
f1$coefficients <- Ridge.cf[names(f1$coefficients),'coef']
tmp_dat <- f_COXPH_predict(all_sets, f1)
tmp_surv <- f_surv_list(tmp, tau = 12*3)
library(timeROC)
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
options(repr.plot.width=8, repr.plot.height=8)
tmp <- f_timeROC_one(tmp_dat$GSE54460, 'Risk_Score')
plot(tmp$roc,time=12*1,title="Time dependent ROC",col="#BC3C29FF")        
plot(tmp$roc,time=12*3,add=T,col="#0072B5FF") 
plot(tmp$roc,time=12*5,add=T,col="#E18727FF") 
legend("bottomright",c(paste("AUC of 1-year =",tmp$auc1),
                 paste("AUC of 3-year =",tmp$auc2),
                 paste("AUC of 5-year =",tmp$auc3)),
       col=c("#BC3C29FF","#0072B5FF","#E18727FF"),lty=1,lwd=2)
```