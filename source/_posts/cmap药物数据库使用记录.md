---
title: Cmap药物数据库使用记录
tags: []
id: '2004'
categories:
  - - uncategorized
date: 2022-09-03 16:59:01
---

Connectivity Map（Cmap）为一个基因表达数据库，由哈佛、剑桥大学和麻省理工学院研究人员构建，利用不同干扰物（包括小分子）处理人类细胞后的基因表达差异，建立一个干扰物、基因表达和疾病相互关联的生物应用数据库。研究团队认为以基因表达谱所建立的基因、疾病与药物的关联性，可协助研究者在药物研发领域上，快速利用基因表达谱数据比对出与疾病高相关性的药物、推论大部分药物分子的主要结构，并能够归纳出药物分子可能的作用机制。（知乎 [FrontScience科研](https://www.zhihu.com/people/gpf-64)）

## 导出差异基因

```R
x <- readRDS('DEG_filer.rds')
up <- subset(x, log2FoldChange > 1)
up <- head(up, 150)
down <- subset(x, log2FoldChange < -1)
down <- tail(down, 150)
up <- unique(up$gene_name)
down <- unique(down$gene_name)
write.table(x = as.data.frame(up), file = 'cmap_input_up.txt', row.names = F, quote = F)
write.table(x = as.data.frame(down), file = 'cmap_input_down.txt', row.names = F, quote = F)
```

## 进行Cmap分析

![](https://img.limour.top/archives_2023/2022/09/03/63133cc60e6e1.webp)

[点此进入分析页面](https://clue.io/query)，完成后可下载tar.gz的结果，解压后arfs/TAG下面的gct是我们需要的

## 对结果进行可视化

### 读取数据

```R
cmap <- read.table('query_result.gct', header = T, sep = '\t', allowEscapes = T, quote = '', comment.char = '#', skip=2)
cmap <- cmap[-1,]
saveRDS(cmap, 'cmap_query_result.rds')
```

*   pert\_iname  干扰物命名(化合物或者基因名)
*   cell\_iname 细胞系名
*   pert\_type 干扰类型(化合物/基因敲除、过表达)
*   pert\_idose 剂量
*   pert\_itime 时间
*   nsample 样本数
*   cc\_q75 重复相同实验的结果相似度指标
*   ss\_ngene 干扰影响强度指标
*   tas 综合cc\_q75与ss的评价指标结果
*   raw\_cs 连通性分数

### 清洗数据

```R
test <- cmap[c('pert_iname', 'cell_iname', 'tas', 'raw_cs', 'fdr_q_nlog10')]
test$tas <- as.numeric(test$tas)
test$raw_cs <- as.numeric(test$raw_cs)
test$fdr_q_nlog10 <- as.numeric(test$fdr_q_nlog10)
test <- tail(test, 100)
```

### 绘图

#### 点图

```R
library(ggplot2)
options(repr.plot.width=8, repr.plot.height=20)
p <- ggplot(test, aes(x = cell_iname, y = pert_iname))
p <- p + geom_point(aes(size=fdr_q_nlog10,color=raw_cs)) + scale_size(range = c(5,10))
p <- p + scale_colour_gradient2(low=rgb(1,0,0,0.5) ,high=rgb(0,0,1,0.5), mid = 'white')
p <- p + theme_bw() + theme(axis.text=element_text(size=12),
                            axis.title=element_text(size=20), 
                            axis.text.x=element_text(angle=90,hjust = 1,vjust=0.5))
p
```

#### 热图

```R
f_long2wide <- function(df, colN, rowN, valueN){
    rowR <- unique(df[[rowN]])
    colR <- unique(df[[colN]])
    res <- matrix(nrow=length(rowR), ncol=length(colR), data = 0)
    resN <- matrix(nrow=length(rowR), ncol=length(colR), data = 0)
    rownames(res) <- rowR
    colnames(res) <- colR
    rownames(resN) <- rowR
    colnames(resN) <- colR
    for(i in 1:nrow(df)){
        res[df[i,rowN],df[i,colN]] <- res[df[i,rowN],df[i,colN]] + df[i, valueN]
        resN[df[i,rowN],df[i,colN]] <- resN[df[i,rowN],df[i,colN]] + 1
    }
    res <- res / resN
    res
}
```

```R
library(ComplexHeatmap)
library(circlize)
col_fun <- colorRamp2(
  c(-1, 0, 1), 
  c("#BC3C29AA", "white", "#99CCCCAA")
  )
test <- cmap[c('pert_iname', 'cell_iname', 'tas', 'raw_cs', 'fdr_q_nlog10')]
test$fdr_q_nlog10 <- as.numeric(test$fdr_q_nlog10)
test$raw_cs <- as.numeric(test$raw_cs)
test <- subset(test, cell_iname %in% c('LNCAP', 'PC3', 'VCAP'))
mat <- f_long2wide(tail(test, 20), 'cell_iname', 'pert_iname', 'raw_cs')
mat_tmp <- f_long2wide(test, 'cell_iname', 'pert_iname', 'raw_cs')
mat <- mat_tmp[rownames(mat),]
mat[is.nan(mat)]  <- 0
options(repr.plot.width=4, repr.plot.height=5)
Heatmap(mat, col=col_fun, name = 'raw_cs', row_order=nrow(mat):1, column_order = 1:3)
```