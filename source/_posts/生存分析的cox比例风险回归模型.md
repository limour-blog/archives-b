---
title: 生存分析的Cox比例风险回归模型
tags: []
id: '1913'
categories:
  - - uncategorized
date: 2022-07-17 00:27:45
---

如果只研究患者的中位生存时间或限制平均生存时间，或比较某一变量的限制平均生存时间，可以采用[生存分析](https://occdn.limour.top/1949.html)。如果自变量有多个，需要从中寻找主要的影响因素，则可以采用Cox回归模型进行分析。若某一变量的βj=0，则称其对风险函数无贡献。对风险函数无贡献的变量可以从模型中剔除，然后重现计算新的Cox模型，称为用Cox回归分析的**逐步回归方法**对变量进行筛选。（《卫生统计学》赵耐青，下同）

多个自变量之间有可能存在交互作用，比如催化剂的单独效应为零，与其他因素配合却能较大地提高效应。在回归分析中，若，X1、X2之间存在交互作用，最常用的方法是在回归模型中增加这两个变量的乘积项X1X2作为新的自变量，称为**交互作用项**。交互作用项的引入主要根据研究的背景知识。

[Logisitic回归](https://occdn.limour.top/2099.html)和Cox回归均基于大样本的假定，因此所需要的样本含量多于多重线性回归，需要达到**模型自变量个数的15~20倍**。如果样本含量较小，则难以得到稳定可靠的结论。

![](https://img.limour.top/archives_2023/2022/07/17/62d3463eb5034.webp)

## Cox回归的假设

*   HR不随时间变化，满足比例风险假定
*   无“过早死亡”的个体和“活得太久”的个体
*   连续自变量具有线性形式，自变量间无共线性

参考1：[JeremyL](https://www.jianshu.com/p/97180e6cf884)

## 准备资料

```R
group <- t(readRDS('prad_tpm_Cu_death.rds'))
clinical <- read.csv('TCGA-PRAD_clinical.csv', row.names = 'bcr_patient_barcode')
mergeID <- intersect(rownames(clinical), rownames(group))
df <- cbind(group[mergeID,], clinical[mergeID, c('dcf_time', 'dcf_status', 'os_time', 'os_status')])
df[,'dcf_status'] = ifelse(df[,'dcf_status']==1,0,1)
df
```

## Cox回归

```R
library("survival")
library("survminer")
coxmf <- paste0("Surv(dcf_time/365, dcf_status==0)~", paste(colnames(df)[1:10], collapse = '+'))
coxmf
res.cox <- coxph(formula(coxmf), data =  df)
res.cox
tmp_res <- summary(res.cox)
res <- as.data.frame(tmp_res$conf.int)
res <- res[,c(1,3,4)]
colnames(res) <- c('mean', 'lower', 'upper')
res[['Pvalue']] <- tmp_res$coefficients[,'Pr(>z)']
res[['VarName']] <- rownames(res)
res
saveRDS(res, 'Cox_res.rds')
```

## Cox模型进行预测

想要评估GLS对估计生存概率的影响。在本例中，构造了一个有两行的新数据，每一行代表一个GLS值;其他自变量设置为它们的平均值(如果是连续变量)或最低水平(如果是离散变量)。对于一个哑变量，平均值是数据集中编码为1的比例。（[JeremyL](https://www.jianshu.com/p/3f53255f8b60)）

```R
gls_df <- as.data.frame(rbind(colMeans(df[1:10]),colMeans(df[1:10])))
gls_df[['GLS']] <- with(df,c(min(GLS),max(GLS)))
gls_df
ggsurvplot(survfit(res.cox, newdata = gls_df), conf.int = TRUE,,data = df)
```

## [检测是否满足假设](https://www.jianshu.com/p/97180e6cf884)

*   比例风险假设
*   test.ph <- cox.zph(res.cox)
*   ggcoxzph(test.ph)
*   p无统计学意义表示满足假设

*   无离群值
*   ggcoxdiagnostics(res.cox, type = "dfbeta", linear.predictions = F)
*   ggcoxdiagnostics(res.cox, type = "deviance", linear.predictions = F)

*   线性假设
*   ggcoxfunctional(Surv(dcf\_time/365, dcf\_status==0) ~ GLS + log(GLS) + sqrt(GLS), data = df)
*   ggcoxfunctional(formula(coxmf), data = df)
*   拟合线应该是线性的表示满足Cox比例风险模型的假设