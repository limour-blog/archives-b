---
title: Seurat绘制指定基因的小提琴图
tags: []
id: '1550'
categories:
  - - R绘图奇技淫巧
date: 2022-02-09 22:11:30
---

## 第一步 定义绘图函数

```r
f_VlnBoxPlot_gene <- function(scRNAo, groupN, sampleN, geneN, rmZero=F){
    res <- data.frame(geneID=colnames(scRNAo))
    res[['groupN']] = scRNAo[[groupN]][res$geneID,]
    res[['sampleN']] = scRNAo[[sampleN]][res$geneID,]
    res[['value']] = scRNAo@assays$RNA@data[geneN, res$geneID]
    if(rmZero){
        subset(res, value > 0)
    }else{
        res
    }
}
require(ggplot2)
f_VlnBoxPlot_df <- function(df, geneN, groupN='groupN'){
    p <- ggplot(df, aes(x=!!sym(groupN), y=value, fill= !!sym(groupN)))
    p <- p + theme_bw() + NoLegend()
    p <- p + geom_violin() # 绘制小提琴图
    p <- p + stat_ydensity(trim = TRUE, scale = 'width', adjust = 1) # 绘制小提琴图
    p <- p + geom_boxplot(width=0.618) # 绘制箱型图
    p <- p + stat_summary(fun="mean",geom="point",color='white') # 添加均值点
    p <- p + labs(x=NULL, y=NULL) # 删除xy轴标题
    p <- p + scale_y_continuous('', breaks = floor(max(layer_scales(p)$y$range$range))) # 美化y轴刻度
#     p <- p + theme(panel.background = element_rect(colour="black")) # 添加框线
    p <- p + theme(plot.margin = unit(c(0,0,0,0), "cm")) # 尽量删除图外空白
    p <- p + facet_grid(sampleN ~ .) # 添加分页
    p <- p + labs(title = geneN) + theme(plot.title = element_text(hjust = 0.5))
    p <- p + theme(axis.text.x=element_text(hjust = 1, angle = 45))
    p
}
```