---
title: CBNplot推断临床变量对通路的影响
tags: []
id: '2011'
categories:
  - - uncategorized
date: 2022-09-09 07:40:12
---

## 清洗数据

```R
vsted <- readRDS('rininiang.rds')
group <- readRDS('tcga.predict.rds')
incSample <- rownames(group)[group$group == 'High Risk']
pwayGSE  <- readRDS('pwayGSE.rds')
spath <- read.csv('fig5_selected_pGSE.csv', row.names = 1)
pwayGSE@result <- pwayGSE@result[rownames(spath),]
require(org.Hs.eg.db)
set.seed(123)
CBNplot::bnpathplot(results = pwayGSE, exp = vsted, expSample = incSample, R = 200,
           nCategory = 100,
           expRow='ENSEMBL', orgDb=org.Hs.eg.db)
group <- group[colnames(vsted),]
```

## 推断临床变量对通路的调控

```R
bnCov <- CBNplot::bnpathplot(pwayGSE,
                    vsted,
                    nCategory = 1000,
                    adjpCutOff = 0.05,
                    expSample=rownames(group),
                    algo="hc", strType="normal",
                    otherVar=group$group,
                    otherVarName="Risk_Group",
                    R=200, cl=parallel::makeCluster(4),
                    returnNet=T,
                    shadowText=T)
igraph::is.dag(bnlearn::as.igraph(bnCov$av))
bnFit <- bnlearn::bn.fit(bnCov$av, bnCov$df)
bnCov$plot
```

点此查看官方手册的[更进一步的分析](https://noriakis.github.io/software/CBNplot/including-clinical-variables.html#classification-using-bn)