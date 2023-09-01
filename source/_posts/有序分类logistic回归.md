---
title: 有序分类Logistic回归
tags: []
id: '1908'
categories:
  - - uncategorized
date: 2022-07-15 08:36:33
---

## Logistic回归的假设

*   **假设1**：因变量唯一，且为有序多分类变量
*   **假设2**：存在一个或多个自变量，可为连续、有序多分类或无序分类变量。
*   **假设3**：自变量之间无多重共线性。
*   **假设4**：模型满足“比例优势”假设。

## 通过conda安装纯净环境的Logistic分析包

*   conda create -n logistic -c conda-forge r-base=4.1.3
*   conda activate logistic
*   conda install -c conda-forge r-tidyverse=1.3.1 -y
*   conda install -c conda-forge r-irkernel -y
*   conda install -c conda-forge r-foreign -y
*   conda install -c conda-forge r-mass -y
*   conda install -c conda-forge r-hmisc -y
*   conda install -c conda-forge r-reshape2 -y
*   conda install -c r r-car -y
*   Rscript -e "IRkernel::installspec(name='logistic', displayname='r-logistic')"

## 准备数据

```R
group <- t(readRDS('prad_tpm_Cu_death.rds'))
clinical <- read.csv('TCGA-PRAD_clinical.csv', row.names = 'bcr_patient_barcode')
f_TCGA_gleason_grade <- function(primary_gleason_grade, secondary_gleason_grade){
    primary_gleason_grade <- as.numeric(unlist(data.frame(strsplit(primary_gleason_grade, ' '))[2,]))
    secondary_gleason_grade <- as.numeric(unlist(data.frame(strsplit(secondary_gleason_grade, ' '))[2,]))
    primary_gleason_grade + secondary_gleason_grade
}
clinical[['gleason']] <- f_TCGA_gleason_grade(clinical$primary_gleason_grade, clinical$secondary_gleason_grade)
mergeID <- intersect(rownames(clinical), rownames(group))
df <- cbind(group[mergeID,], clinical[mergeID, c('gleason', 'dcf_time', 'dcf_status', 'os_time', 'os_status')])
df[,'dcf_status'] = ifelse(df[,'dcf_status']==1,0,1)
df
```

## 多重共线性检测

*   qr(as.matrix(df\[1:10\]))$rank == 10
*   kappa(cor(as.matrix(df\[1:10\])), exact= TRUE) < 100
*   library(car)
*   vif(lm.sol)

## Logistic回归

```R
require(foreign)
require(ggplot2)
require(MASS)
require(Hmisc)
require(reshape2)
lmf <- paste0("factor(gleason)~", paste(colnames(df)[1:10], collapse = '+'))
lmf
m <- polr(formula(lmf), data = df, Hess = TRUE)
m
drop1(m,test="Chi") 
(ci <- confint(m))
exp(cbind(OR = coef(m), ci))
res <- as.data.frame(exp(cbind(OR = coef(m), ci)))
res[['Pvalue']] <- drop1(m,test="Chi")[rownames(res),'Pr(>Chi)']
res[['VarName']] <- rownames(res)
colnames(res)[1:3] <- c('mean', 'lower', 'upper')
res
saveRDS(res, 'Logistic.rds')
```

## 绘制森林图

[绘制函数来源](https://occdn.limour.top/2097.html)

```R
df <- readRDS('Logistic.rds')
options(repr.plot.width=8, repr.plot.height=6)
f_forestplot(df, zero = 1)
```