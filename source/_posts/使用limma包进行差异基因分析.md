---
title: 使用limma包进行差异基因分析
tags: []
id: '1941'
categories:
  - - uncategorized
date: 2022-07-30 06:36:33
---

虽然现在已经是高通量测序的时代，大家基本都是从counts矩阵出发，使用DESeq2进行[差异表达分析](https://occdn.limour.top/2132.html)，但是[GEO](https://www.ncbi.nlm.nih.gov/geo/)和[ArrayExpress](https://www.ebi.ac.uk/arrayexpress/)上的仍有海量且持续更新的芯片数据，有时候也不可避免遇到一些FPKM格式乃至已经进行了z-score转换的数据，对于这些数据的分析，我们可以认为其在适当变换下(log2FPKM)，满足正态分布，那么仍可以使用limma直接进行分析。下面博主以[E-MEXP-1422](https://occdn.limour.top/2165.html)为例，写一份分析代码的demo。

```R
require(limma)
f_DE_limma <- function(cts_bb, rowInfo, ControlN, TreatN, rm.NA=T, trend=T){
    # trend 表示先验方差是否与基因表达值的大小相关，False表示其为常数
    cts_b <- cts_bb[,c(ControlN, TreatN)]
    conditions <- c(rep("Control",length(ControlN)), rep("Treat",length(TreatN)))
    design <- model.matrix(~0+factor(conditions))
    colnames(design) <- levels(factor(conditions))
    rownames(design) <- colnames(cts_b)
    print(design)
    contrast.matrix <- makeContrasts('Treat-Control', levels = design)
    print(contrast.matrix)
    fit <- lmFit(cts_b, design) # 拟合线性模型
    fit2 <- contrasts.fit(fit, contrast.matrix) # 计算拟合系数和标准误差
    fit2 <- eBayes(fit2, trend=trend) # 通过经验贝叶斯方法估计统计量和logFC值
    tempOutput <- topTable(fit2, coef=1, n=Inf)
    if(!is.null(rowInfo)){tempOutput <- cbind(rowInfo[rownames(tempOutput),], tempOutput)}
    if(rm.NA){tempOutput <- na.omit(tempOutput)}
    tempOutput
}
# 经过 oligo::rma 标准化后提取出来的表达矩阵
data.exprs
# SDRF <- read.delim('E-MEXP-1422.sdrf.txt') 
# 从sdrf文件可知 AF15、AF16为PROX1 siRNA组
# AF6、AF14为GFP siRNA组
Ct1 <- c('AF6.CEL', 'AF14.CEL')
Tt1 <- c('AF15.CEL', 'AF16.CEL')
f_DE_limma(data.exprs, NULL, Ct1, Tt1, F)
```