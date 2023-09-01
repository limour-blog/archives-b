---
title: scMetabolism比较绘图
tags: []
id: '1564'
categories:
  - - R绘图奇技淫巧
date: 2022-02-10 02:46:51
---

## 定义绘图函数

```r
f_metaG2G <- function(metaG, matrixN=F){
    res  <- list()
    alltype <- unique(metaG[[1]])
    for(type in alltype){
        res[[type]] <- rownames(metaG)[metaG[[1]] == type]
        if (matrixN){
            res[[type]] <- gsub('-','.',res[[type]])
        }
    }
    res
}
f_matrix_groupMean <- function(matrixA, group, matrixN = T){
    res <- data.frame(row.names = rownames(matrixA))
    group <- f_metaG2G(group, matrixN = matrixN)
    for(name in names(group)){
        if (length(group[[name]]) == 1){
            res[[name]] <- matrixA[,group[[name]]]
        }else{
            res[[name]] <- rowMeans(matrixA[,group[[name]]])
        }
    }
    res
}
require(reshape2)
require(ggplot2)
f_matrix_heatmap <- function(dfA, levels = NULL){
    # 转换前，先增加一列ID列，保存行名字
    dfA$df_ID <- rownames(dfA)
    dfm <- melt(dfA, na.rm = T, id.vars = c('df_ID'))
    dfm$variable <- factor(x = as.character(dfm$variable), ordered = T)
    if (length(levels) > 0){
        dfm$df_ID  <- factor(x = as.character(dfm$df_ID), levels = rev(levels))
    }
    p <- ggplot(dfm, aes(x=variable, y=df_ID)) 
    p <- p + geom_tile(aes(fill=value))
#     p <- p + scale_fill_gradient(low = 'white', high = 'red')
    p <- p + scale_fill_gradientn(colours = c('#3E5CC5','#65B48E','#E6EB00','#E64E00'))
#     p <- p + scale_color_distiller(palette = "Spectral")
    p <- p + xlab("samples") + theme_bw() + theme(panel.grid.major = element_blank()) + theme(legend.key=element_blank())
    p <- p + theme(axis.text.x=element_text(angle=45, hjust=1, vjust=1))
    p <- p + labs(x=NULL, y=NULL) # 删除xy轴标题 
    p
}
```

## 绘图

```r
# 01.组合metadata
immune[['tmp']] <- paste(as.character(immune[['immune_type_2']][[1]]), as.character(immune[['group']][[1]]))
# 02.切分感兴趣的细胞群并计算代谢强度
M_c <- subset(immune, cell_type == 'Myeloid')
M_c <- sc.metabolism.Seurat(obj = M_c, method = "VISION", imputation = F, ncores = 12, metabolism.type = "KEGG")
#03.读入感兴趣的通路
sM_h <- read.csv('selected_pathway_high.CSV', header = F)
sM_m <- read.csv('selected_pathway_medium.CSV', header = F)
sM_l <- read.csv('selected_pathway_low.CSV', header = F)
#04.调用绘图函数
P7_h <- f_matrix_heatmap(f_matrix_groupMean(M_c@assays$METABOLISM$score[unlist(sM_h),], M_c[['tmp']]), levels = unlist(sM_h)) + theme(axis.text.x=element_blank())
P7_m <- f_matrix_heatmap(f_matrix_groupMean(M_c@assays$METABOLISM$score[unlist(sM_m),], M_c[['tmp']]), levels = unlist(sM_m)) + theme(axis.text.x=element_blank())
P7_l <- f_matrix_heatmap(f_matrix_groupMean(M_c@assays$METABOLISM$score[unlist(sM_l),], M_c[['tmp']]), levels = unlist(sM_l))
#05.组合绘图
require(patchwork)
require(dplyr)
P7 <- (P7_h / P7_m / P7_l) + plot_layout(nrow = 3, heights = c(nrow(sM_h), nrow(sM_m), nrow(sM_l) + 3))
#06。保存绘图结果
ggsave(P7, filename = 'HSPC vs CRPC_M.pdf', dpi = 1200, width = 12, height = 10, device = 'pdf', limitsize = FALSE)
```