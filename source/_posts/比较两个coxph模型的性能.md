---
title: 比较两个COXPH模型的性能
tags: []
id: '1963'
categories:
  - - uncategorized
date: 2022-08-10 16:25:34
---

## 安装补充包

*   [conda activate ggsurvplot](https://occdn.limour.top/2213.html)

*   install.packages('survIDINRI')

## 构建两个COXPH模型

### 准备数据

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
library("scales")
library(nomogramFormula)
library(survIDINRI)
df1 <- readRDS('df.rds')
df2 <- readRDS('clinical.rds')
df <- cbind(df1[rownames(df2),1:11],df2)
```

### 选择配色

```R
pal_nejm("default")(8)
show_col(pal_nejm("default")(8)) #挑选配色
```

### 计算某风险指标

```R
coxmf1 <- paste0("Surv(time, status==1)~", paste(colnames(df)[1:11], collapse = '+'))
coxmf1
f1 <- cph(formula(coxmf1), data=df1, x=T, y=T, surv = T) # 构建COX比例风险模型
surv1 <- Survival(f1) # 构建生存概率函数
dd=datadist(df1)
options(datadist="dd") 
nom1 <- nomogram(f1, fun=list(function(x) {surv1(1*12, x)},
                            function(x) {surv1(3*12, x)},
                            function(x) {surv1(5*12, x)}),
                             funlabel=c("1-year Disease-free Probability",
                                        "3-year Disease-free Probability",
                                        "5-year Disease-free Probability"))
 
options(repr.plot.width=12, repr.plot.height=8)
plot(nom1, xfrac=.2)
results <- formula_rd(nomogram = nom1)
df$f1_points <- points_cal(formula = results$formula,rd=df)
df
years1 <- timeROC(T=df$time,delta=df$status,
                      marker=df$f1_points,cause=1,
                      weighting="marginal",
    #                   other_markers=as.matrix(df$`1-year`),
                      times=12*c(1,3,5), ROC=T, iid=T)
years1
auc1.ci <- confint(years1, parm=NULL, level = 0.95,n.sim=2000)
auc1.ci <- auc1.ci$CI_AUC
auc1.ci
years1.auc1 <- paste0(round(years1$AUC[1],4)*100,'(',auc1.ci[1,1],'~',auc1.ci[1,2],')%')
years1.auc2 <- paste0(round(years1$AUC[2],4)*100,'(',auc1.ci[2,1],'~',auc1.ci[2,2],')%')
years1.auc3 <- paste0(round(years1$AUC[3],4)*100,'(',auc1.ci[3,1],'~',auc1.ci[3,2],')%')
options(repr.plot.width=8, repr.plot.height=8)
plot(years1,time=12*1,title="Time dependent ROC",col="#BC3C29FF")        
plot(years1,time=12*3,add=TRUE,col="#0072B5FF") 
plot(years1,time=12*5,add=TRUE,col="#E18727FF") 
legend("bottomright",c(paste("AUC of 1-year =",years1.auc1),
                 paste("AUC of 3-year =",years1.auc2),
                 paste("AUC of 5-year =",years1.auc3)),
       col=c("#BC3C29FF","#0072B5FF","#E18727FF"),lty=1,lwd=2)
```

### 计算第一个模型

```R
dd=datadist(df)
options(datadist="dd") 
coxmf2 <- paste0("Surv(time, status==1)~PSA+Age+T2vs3+N0vs1+GS")
coxmf2
f2 <- cph(formula(coxmf2), data=df, x=T, y=T, surv = T) # 构建COX比例风险模型
surv2 <- Survival(f2) # 构建生存概率函数
nom2 <- nomogram(f2, fun=list(function(x) {surv2(1*12, x)},
                            function(x) {surv2(3*12, x)},
                            function(x) {surv2(5*12, x)}),
                             funlabel=c("1-year Disease-free Probability",
                                        "3-year Disease-free Probability",
                                        "5-year Disease-free Probability"))
 
options(repr.plot.width=12, repr.plot.height=8)
plot(nom2, xfrac=.2)
results <- formula_rd(nomogram = nom2)
df$f2_points <- points_cal(formula = results$formula,rd=df)
df
years2 <- timeROC(T=df$time,delta=df$status,
                      marker=df$f2_points,cause=1,
                      weighting="marginal",
    #                   other_markers=as.matrix(df$`1-year`),
                      times=12*c(1,3,5), ROC=T, iid=T)
years2
auc2.ci <- confint(years2, parm=NULL, level = 0.95,n.sim=2000)
auc2.ci <- auc2.ci$CI_AUC
auc2.ci
years2.auc1 <- paste0(round(years2$AUC[1],4)*100,'(',auc2.ci[1,1],'~',auc2.ci[1,2],')%')
years2.auc2 <- paste0(round(years2$AUC[2],4)*100,'(',auc2.ci[2,1],'~',auc2.ci[2,2],')%')
years2.auc3 <- paste0(round(years2$AUC[3],4)*100,'(',auc2.ci[3,1],'~',auc2.ci[3,2],')%')
options(repr.plot.width=8, repr.plot.height=8)
plot(years2,time=12*1,title="Time dependent ROC",col="#BC3C29FF")        
plot(years2,time=12*3,add=TRUE,col="#0072B5FF") 
plot(years2,time=12*5,add=TRUE,col="#E18727FF") 
legend("bottomright",c(paste("AUC of 1-year =",years2.auc1),
                 paste("AUC of 3-year =",years2.auc2),
                 paste("AUC of 5-year =",years2.auc3)),
       col=c("#BC3C29FF","#0072B5FF","#E18727FF"),lty=1,lwd=2)
```

### 计算第二个模型

```R
coxmf3 <- paste0("Surv(time, status==1)~PSA+Age+T2vs3+N0vs1+GS+f1_points")
coxmf3
res.cox <- coxph(formula(coxmf3), data =  df)
tmp_res <- summary(res.cox)
tmp_res
f3 <- cph(formula(coxmf3), data=df, x=T, y=T, surv = T) # 构建COX比例风险模型
surv3 <- Survival(f3) # 构建生存概率函数
nom3 <- nomogram(f3, fun=list(function(x) {surv3(1*12, x)},
                            function(x) {surv3(3*12, x)},
                            function(x) {surv3(5*12, x)}),
                             funlabel=c("1-year Disease-free Probability",
                                        "3-year Disease-free Probability",
                                        "5-year Disease-free Probability"))
 
options(repr.plot.width=12, repr.plot.height=8)
plot(nom3, xfrac=.2)
results <- formula_rd(nomogram = nom3)
df$f3_points <- points_cal(formula = results$formula,rd=df)
df
years3 <- timeROC(T=df$time,delta=df$status,
                      marker=df$f3_points,cause=1,
                      weighting="marginal",
    #                   other_markers=as.matrix(df$`1-year`),
                      times=12*c(1,3,5), ROC=T, iid=T)
years3
auc3.ci <- confint(years3, parm=NULL, level = 0.95,n.sim=2000)
auc3.ci <- auc3.ci$CI_AUC
auc3.ci
years3.auc1 <- paste0(round(years3$AUC[1],4)*100,'(',auc3.ci[1,1],'~',auc3.ci[1,2],')%')
years3.auc2 <- paste0(round(years3$AUC[2],4)*100,'(',auc3.ci[2,1],'~',auc3.ci[2,2],')%')
years3.auc3 <- paste0(round(years3$AUC[3],4)*100,'(',auc3.ci[3,1],'~',auc3.ci[3,2],')%')
options(repr.plot.width=8, repr.plot.height=8)
plot(years3,time=12*1,title="Time dependent ROC",col="#BC3C29FF")        
plot(years3,time=12*3,add=TRUE,col="#0072B5FF") 
plot(years3,time=12*5,add=TRUE,col="#E18727FF") 
legend("bottomright",c(paste("AUC of 1-year =",years3.auc1),
                 paste("AUC of 3-year =",years3.auc2),
                 paste("AUC of 5-year =",years3.auc3)),
       col=c("#BC3C29FF","#0072B5FF","#E18727FF"),lty=1,lwd=2)
```

## 比较ROC曲线

```R
options(repr.plot.width=8, repr.plot.height=8)
plot(years2,time=12*1,title="Time dependent ROC",col="#20854EFF")    
plot(years3,time=12*1,add=T,col="#BC3C29FF") 
plot(years2,time=12*3,add=TRUE,col="#7876B1FF") 
plot(years3,time=12*3,add=T,col="#0072B5FF") 
plot(years2,time=12*5,add=TRUE,col="#6F99ADFF") 
plot(years3,time=12*5,add=T,col="#E18727FF") 
legend("bottomright",c(paste("Old AUC of 1-year =",years2.auc1),
                 paste("New AUC of 1-year =",years3.auc1),
                 paste("Old AUC of 3-year =",years2.auc2),
                 paste("New AUC of 3-year =",years3.auc2),
                 paste("Old AUC of 5-year =",years2.auc3),
                 paste("New AUC of 5-year =",years3.auc3)),
       col=c("#20854EFF","#BC3C29FF","#7876B1FF","#0072B5FF","#6F99ADFF","#E18727FF"),lty=1,lwd=2)
compare(years2,years3,adjusted=T)
```

## 计算NRI和IDI

*   NRI = (c1-b1)/N1 + (b2-c2)/N2
*   a1表示在患病组中新模型和旧模型都分类正确的样本数，b1则表示旧模型分类正确而新模型分类错误，b1越小越好，c1反映了从旧模型到新模型预测的改善情况，c1越大表示改进越多，同理计算出非患病组的a2、b2、c2、d2数值。
*   当NRI>0,则为正改善，表示新模型较旧模型对事件的预测能力更好；
*   IDI = (Pnew,events - Pold,events) - (Pnew,non-events - Pold,non-events)
*   其中Pnew,events、Pold,events表示在患者组中，新模型和旧模型对于每个个体预测疾病发生概率的平均值，两者相减表示预测概率提高的变化量，对于患者来说，预测患病的概率越高，模型越准确，因此差值越大则提示新模型越好。
*   而Pnew,non-events、Pold,non-events表示在非患者组中，新模型和旧模型对于每个个体预测疾病发生概率的平均值，两者相减表示预测概率减少的量，对于非患者来说，预测患病的概率越低，模型越准确，因此差值越小则提示新模型越好。
*   与NRI类似，若IDI>0，则为正改善，说明新模型比旧模型的预测能力有所改善；

```R
covs0 <- as.matrix(df[c('PSA','Age','T2vs3','N0vs1','GS')])
covs1 <- as.matrix(df[c('PSA','Age','T2vs3','N0vs1','GS','f1_points')])
set.seed(0)
t1=12*3
res.IDI <- IDI.INF(df[c('time', 'status')], covs0, covs1, t1, npert=1000)
IDI.INF.OUT(res.IDI)
# M1表示IDI
# M2表示NRI
# M3表示中位数差异
IDI.INF.GRAPH(res.IDI)
## M1 red area; M2 distance between black points; M3 distance between gray points
```

## 绘制临床决策曲线DCA

*   DCA曲线，即决策曲线分析法（Decision Curve Analysis）,是用来帮助确定高风险患者进行干预，而低风险患者避免干预（避免过度医疗），即评价患者获益程度的一种评估方法。
*   图中有两条虚线，横着的那条虚线表示所有样本均不进行干预，净获益为0，斜的虚线表示所有样本均进行干预。
*   与两条虚线有交叉的模型没有临床价值，在上面的曲线临床价值比在下面的曲线大

```R
dca_p <- dca(f2, f3, times=12*c(1, 3, 5))
options(repr.plot.width=12, repr.plot.height=6)
ggplot(dca_p) + scale_color_nejm()
```