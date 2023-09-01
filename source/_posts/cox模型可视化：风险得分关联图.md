---
title: COX模型可视化：风险得分关联图
tags: []
id: '1967'
categories:
  - - uncategorized
date: 2022-08-11 12:30:55
---

## 安装补充包

*   [conda activate ggsurvplot](https://occdn.limour.top/2223.html)
*   \# conda install -c conda-forge r-ggplotify -y
*   conda install -c conda-forge r-circlize -y
*   conda install -c bioconda bioconductor-complexheatmap -y
*   install.packages('ggrisk')

## 绘制风险得分关联图

```R
library(ggrisk)
ggrisk(f1,
       code.highrisk = 'High Risk',#高风险标签，默认为 ’High’
       code.lowrisk = 'Low Risk', #低风险标签，默认为 ’Low’
       title.A.ylab='Risk Score', #A图 y轴名称
       title.B.ylab='Survival Time(month)', #B图 y轴名称，注意区分year month day
       title.A.legend='Risk Group', #A图图例名称
       title.B.legend='Status',     #B图图例名称
       title.C.legend='Relative Expression', #C图图例名称
       relative_heights=c(0.1,0.1,0.01,0.15), #A、B、热图注释和热图C的相对高度    
       color.A=c(low='#99CCFFAA',high='#FF6666AA'),#A图中点的颜色
       color.B=c(code.0='#99CCFFAA',code.1='#FF6666AA'), #B图中点的颜色
       color.C=c(low='#99CCFFAA',median='white',high='#FF6666FF'), #C图中热图颜色
       vjust.A.ylab=1, #A图中y轴标签到y坐标轴的距离,默认是1
       vjust.B.ylab=2  #B图中y轴标签到y坐标轴的距离,默认是2
       )
```

## 绘制生存曲线

```R
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
```

```R
df$group <- 'Low Risk'
df$group[df$f1_points > median(df$f1_points)] <- 'High Risk'
df$group <- as.factor(df$group)
test <- f_surv(df, tau = 12*3, risk.table = F, ncensor.plot=F)
options(repr.plot.width=6, repr.plot.height=3)
test$plot
```

## 绘制ComplexHeatmap

更详细的绘制见知乎答主：[生信学习手册](https://zhuanlan.zhihu.com/p/372130351)

### 准备数据

```R
dfc <- readRDS('clinical.rds')
dfc <- cbind(df1[rownames(dfc),1:11],dfc)
dfc$group <- df$group
results <- formula_rd(nomogram = nom1)
dfc$Risk_Points <- points_cal(formula = results$formula,rd=dfc)
dfc
saveRDS(dfc, 'dfc.rds')
```

### 绘图

```R
dfc <- readRDS('clinical.rds')
dfc <- cbind(df1[rownames(dfc),1:11],dfc)
dfc$group <- df$group
results <- formula_rd(nomogram = nom1)
dfc$Risk_Points <- points_cal(formula = results$formula,rd=dfc)
dfc
saveRDS(dfc, 'dfc.rds')

library(circlize)
library(ComplexHeatmap)
dfc <- dfc[order(dfc$Risk_Points),]
#定义注释信息
ha <- HeatmapAnnotation(`Risk group` = dfc$group,
                        `Risk Points` = dfc$Risk_Points,
                        DFS.status = dfc$status,
                        DFS.time = dfc$time,
                        T = dfc$T,
                        N = dfc$N,
                        M = dfc$M,
                        logPSA = dfc$PSA,
                        `Gleason Score` = dfc$GS,
                        Age = dfc$Age,
                        col = list(
                            `Risk group` = c("Low Risk" = "#99CCFFAA", 'High Risk' = '#FF6666AA')
                        ),
#                         annotation_name_gp = gpar(fontsize = 14),
                        show_annotation_name = TRUE)
mat <- as.matrix(dfc[1:11])
rownames(mat) <- NULL
mat <- t(mat)
col_fun <- colorRamp2(
  c(-4, 0, 4), 
  c("#99CCCCAA", "white", "#BC3C29AA")
  )
options(repr.plot.width=12, repr.plot.height=8)
Heatmap(mat, name = 'Relative Expression', top_annotation = ha, column_order = order(dfc$Risk_Points),
        col=col_fun)
```