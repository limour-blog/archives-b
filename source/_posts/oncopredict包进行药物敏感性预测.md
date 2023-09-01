---
title: oncoPredict包进行药物敏感性预测
tags: []
id: '2005'
categories:
  - - uncategorized
date: 2022-09-04 11:26:50
---

## 安装包

*   conda create -n oncoPredict -c conda-forge r-base=4.1.3
*   conda activate oncoPredict
*   conda install -c conda-forge r-tidyverse -y
*   conda install -c conda-forge r-irkernel -y
*   Rscript -e "IRkernel::installspec(name='oncoPredict', displayname='r-oncoPredict')"
*   conda install -c conda-forge r-nloptr -y
*   conda install -c conda-forge r-lme4 -y
*   conda install -c conda-forge r-pbkrtest -y
*   conda install -c conda-forge r-car -y
*   conda install -c conda-forge r-biocmanager -y
*   conda install -c conda-forge r-ggpubr -y
*   conda install -c bioconda bioconductor-maftools -y
*   BiocManager::install('oncoPredict')
*   从[oncoPredict作者提供的地址](https://osf.io/c6tfx/)下载整理好的[CTRP](http://portals.broadinstitute.org/ctrp.v2.1/)和[GDSC](https://www.cancerrxgene.org/)，[生信技能树的介绍](https://mp.weixin.qq.com/s?__biz=MzAxMDkxODM1Ng==&mid=2247507359&idx=1&sn=e1b1602338792b6bbd7283bbcc07fe81&scene=21#wechat_redirect)。[镜像](https://share.limour.top/d/data/oncoPredict/DataFiles.zip)
*   unzip DataFiles.zip

## 读入训练数据

```R
library(oncoPredict)
CTRP2 <- readRDS('../../../oncoPredict/DataFiles/DataFiles//Training Data/CTRP2_Expr (TPM, not log transformed).rds')
CTRP2 <- log10(CTRP2+1)
GDSC2_Res <- readRDS('../../../oncoPredict/DataFiles/DataFiles//Training Data/CTRP2_Res.rds')
GDSC2_Res <- exp(GDSC2_Res)
```

## 读入预测数据

[f\_dedup\_IQR](https://occdn.limour.top/2157.html)

```R
load('../../../DEG/TCGA/PRAD_tp.rda')
tpm <- data@assays@data$tpm_unstrand
colnames(tpm) <- data@colData$patient
tpm <- tpm[,f_rm_duplicated(colnames(tpm))]
geneInfo <- as.data.frame(data@rowRanges)[c('gene_id','gene_type','gene_name')]
tpm <- f_dedup_IQR(as.data.frame(tpm), geneInfo$gene_name)
comm <- intersect(rownames(CTRP2), rownames(tpm))
CTRP2 <- CTRP2[comm,]
tpm <- tpm[comm,]
tpm <- log10(tpm+1)
```

## 进行预测

```R
library(oncoPredict)
load('oncoPredict_calcPhenotype.rdata')
keep <- rowSums(CTRP2) > 0.8*ncol(CTRP2)
calcPhenotype(trainingExprData = CTRP2[keep,],
              trainingPtype = GDSC2_Res,
              testExprData = as.matrix(tpm),
              batchCorrect = 'eb',  #   "eb" for ComBat  
              powerTransformPhenotype = TRUE,
              removeLowVaryingGenes = 0.2,
              minNumSamples = 10, 
              printOutput = TRUE, 
              removeLowVaringGenesFrom = 'rawData')
```

*   save(CTRP2, GDSC2\_Res, tpm, file = 'oncoPredict\_calcPhenotype.rdata')
*   nano oncoPredict\_calcPhenotype.R
*   conda activate oncoPredict
*   Rscript ./oncoPredict\_calcPhenotype.R --max-ppsize=500000

## 预测结果可视化

### 读入预测结果

```R
testPtype <- read.csv('./calcPhenotype_Output/DrugPredictions.csv', row.names = 1)
testPtype <- log(testPtype)
testPtype
```

### 贴上分组

```R
group <- readRDS('../fig5/tcga.predict.rds')
df <- cbind(testPtype[rownames(group), c('CIL55', 'BRD4132')],group$group)
colnames(df)[[ncol(df)]]  <- 'Risk Group'
df
```

### 绘图

```R
library(ggpubr)
options(repr.plot.width=4, repr.plot.height=4)
my_comparisons <- list(c("Low Risk", "High Risk"))
ggviolin(df, x="Risk Group", y="CIL55", fill = "Risk Group", 
palette = c("#00AFBB", "#E7B800"), 
add = "boxplot", add.params = list(fill="white"))+ 
stat_compare_means(comparisons = my_comparisons, label = "p.signif")#label这里表示选择显著性标记（星号） 
```