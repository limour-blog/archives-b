---
title: COXPH可视化
tags: []
id: '1960'
categories:
  - - uncategorized
date: 2022-08-10 01:28:04
---

## 建模

```R
library(rms)
library(survival)
dat <- readRDS('../../../COXPH/data/PRAD.rds')
pfg <- readRDS('../A_ref_A_fiig.3_A_B/pfg.rds')
y <- Surv(round(dat$meta$pfs_time,2), dat$meta$pfs_status == 1)
y <- data.matrix(y)
x <- dat$data[,pfg]
x <- as.matrix(x)
df <- cbind(x,y)
df <- as.data.frame(df)
dd=datadist(df)
options(datadist="dd") 
coxmf <- paste0("Surv(time, status==1)~", paste(colnames(df)[1:5], collapse = '+'))
coxmf
f2 <- cph(formula(coxmf), data=df, x=T, y=T, surv = T) # 构建COX比例风险模型
```

## Nomo图展示回归系数

```R
surv <- Survival(f2) # 构建生存概率函数
nom <- nomogram(f2, fun=list(function(x) surv(5*12, x),
                            function(x) surv(10*12, x)),
                             funlabel=c("5-year Disease-free Probability",
                                        "10-year Disease-free Probability"))
options(repr.plot.width=12, repr.plot.height=6)
plot(nom, xfrac=.2)
pdf(file = 'nomo.pdf', width = 12, height = 6);plot(nom, xfrac=.2);dev.off()
```

## 森林图展示风险比

```R
res.cox <- coxph(formula(coxmf), data =  df)
tmp_res <- summary(res.cox)
res <- as.data.frame(tmp_res$conf.int)
res <- res[,c(1,3,4)]
colnames(res) <- c('mean', 'lower', 'upper')
res[['Pvalue']] <- tmp_res$coefficients[,'Pr(>z)']
res[['VarName']] <- rownames(res)
res
saveRDS(res, 'res.rds')
```

```R
require(forestplot)
f_forestplot <- function(df, xlab="XR", zero=0, lineheight=unit(10,'mm'), colgap=unit(2,'mm'), graphwidth=unit(60,'mm'), title="Forestplot"){
    df_labeltext <- df[,c('VarName', 'Pvalue')]
    df_labeltext[[paste0(xlab,'(95%CI)')]] <- paste0(sprintf("%0.2f", df$mean),'(',sprintf("%0.2f", df$lower),'~',sprintf("%0.2f", df$upper),')')
    df_labeltext[['Pvalue']] <- sprintf('%0.1e', df_labeltext[['Pvalue']])
    df_labeltext <- rbind(colnames(df_labeltext), df_labeltext)
    df <- rbind(rep(NaN, ncol(df)), df)
    forestplot(labeltext=as.matrix(df_labeltext[,c(1,3,2)]),
               mean=df$mean,
               lower=df$lower,
               upper=df$upper,
               zero=zero,
               boxsize=0.5,
               lineheight=lineheight,
               colgap=colgap,
               graphwidth=graphwidth,
               lwd.zero=2,
               lwd.ci=2, 
               col=fpColors(box='#458B00',
                            summary='#8B008B',
                            lines = 'black',
                            zero = '#7AC5CD'),
               xlab=xlab,
               lwd.xaxis =2,
               txt_gp = fpTxtGp(ticks = gpar(cex = 0.85),
                                xlab  = gpar(cex = 0.8),
                                cex = 0.9),
               lty.ci="solid",
               title=title, 
               line.margin = 1,
               graph.pos=2)
}
res <- readRDS('res.rds')
options(repr.plot.width=6, repr.plot.height=4)
p <- f_forestplot(res, zero = 1, xlab = 'HR')
p
```