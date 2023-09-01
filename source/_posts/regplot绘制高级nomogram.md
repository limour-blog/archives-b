---
title: regplot绘制高级nomogram
tags: []
id: '1966'
categories:
  - - uncategorized
date: 2022-08-11 03:31:45
---

## 安装补充包

*   [conda activate ggsurvplot](https://occdn.limour.top/2215.html)
*   install.packages('regplot')

## 高级nomogram

```R
library(rms)
library("survival")
library("survminer")
library(regplot)
mycol<-c("#A6CEE3","#1F78B4","#33adff","#2166AC")
names(mycol) = c("dencol","boxcocl","obscol","spkcol")
mycol<- as.list(mycol)
dd=datadist(df)
options(datadist="dd") 
coxmf <- paste0("Surv(time, status==1)~", paste(colnames(df)[1:8], collapse = '+'))
coxmf
f1 <- cph(formula(coxmf), data=df, x=T, y=T, surv = T) # 构建COX比例风险模型
```

```R
options(repr.plot.width=12, repr.plot.height=12)
{pdf("nomogram_new.pdf");{
print(dev.list())
  dev.set(2)
  regplot(f1,
#         observation=df[1,], #也可以不展示
        failtime = 12*c(1,3,5), #预测1年、3年和5年的死亡风险
        prfail = TRUE, #cox回归中需要TRUE
        showP = T, #是否展示统计学差异
        droplines = F,#观测2示例计分是否画线
        colors = mycol, #用前面自己定义的颜色
        interval="confidence", #展示观测的可信区间
        clickable=F) 
print(dev.list())
graphics.off()
}}
```

博主PS：这个regplot的绘图输出好奇怪，不知道为啥这样写可以被pdf捕获

## 重抽样校正曲线

```R
set.seed(0)
cal_0 <- calibrate(f1, u=1*12, cmethod='KM', method='boot',  m=98, B=1000)
set.seed(0)
cal_1 <- calibrate(f1, u=3*12, cmethod='KM', method='boot',  m=98, B=1000)
set.seed(0)
cal_2 <- calibrate(f1, u=5*12, cmethod='KM', method='boot',  m=98, B=1000)
```

```R
pdf(file = 'calibrate.pdf', width = 6, height = 6);{plot(cal_0,lwd=2,lty=1,  ##设置线条宽度和线条类型
     errbar.col='#0072B5AA', ##设置一个颜色
     xlab='Nomogram-Predicted Probability of DFS',#便签
     ylab='Actual DFS(proportion)',#标签
     col='#BC3C29AA',#设置一个颜色
     xlim = c(0.1,1),ylim = c(0.1,1),##x轴和y轴范围
     mgp = c(2, 1, 0))
plot(cal_1,lwd=2,lty=1,  ##设置线条宽度和线条类型
     errbar.col='#20854EAA', ##设置一个颜色
     col='#E18727AA',#设置一个颜色
     add = T,
     xlim = c(0.1,1),ylim = c(0.1,1),##x轴和y轴范围
     mgp = c(2, 1, 0)) #控制坐标轴的位置
plot(cal_2,lwd=2,lty=1,  ##设置线条宽度和线条类型
     errbar.col='#6F99ADAA', ##设置一个颜色
     col='#EE4C97AA',#设置一个颜色
     add = T,
     xlim = c(0.1,1),ylim = c(0.1,1),##x轴和y轴范围
     mgp = c(2, 1, 0))
legend("bottomleft",c('1-year', '3-year', '5-year'),
       col=c("#BC3C29AA","#E18727AA","#EE4C97AA"),lty=1,lwd=2)};dev.off()
```

## 导出nomogram的风险预测值

```R
df$Risk_Score <- predict(f1,type="lp",newdata=df)
df
```