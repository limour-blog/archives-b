---
title: Seurat绘制小提琴图与箱型图的叠加图
tags: []
id: '1428'
categories:
  - - R绘图奇技淫巧
  - - 生物信息学
date: 2022-01-19 05:18:35
---

## 第一步 准备测试数据

```r
library(Seurat)
library(SeuratData)
library(ggplot2)
library(patchwork)
library(dplyr)
InstallData('pbmc3k')
pbmc3k
Idents(pbmc3k) <- pbmc3k[['seurat_annotations']]
```

## 第二步 尝试拼图

```r
require(ggplot2)
require(ggfun)
require(ggplotify)
require(aplot)
require(cowplot)
f_VlnBoxPlot_0 <- function(scRNAo, feature, ...){
    vp <- VlnPlot(scRNAo, features = feature, slot = "counts", log = TRUE, ...) + NoLegend() # 获取数据
    colnames(vp$data) <- c('GENE', 'ident') # 统一mapping的名称
    p <- ggplot(vp$data, aes(x=ident,y=GENE,fill=ident)) + vp$theme # 重构绘图
    p <- p + geom_violin() + scale_y_continuous(trans = "log10") # 绘制小提琴图，对y轴进行log10缩放
    p <- p + stat_ydensity(trim = TRUE, scale = 'width', adjust = 1) # 绘制小提琴图
    p <- p + geom_boxplot(width=0.618) # 绘制箱型图
    p <- p + stat_summary(fun="mean",geom="point",color='white') # 添加均值点
    p <- p + theme(plot.title=element_blank()) # 删除总标题
    p <- p + labs(x=NULL, y=feature) # 删除x轴标题，y标题改成基因名
    p <- p + theme(axis.title.y=element_text(vjust=0.5,angle=0)) # y标题垂直居中、水平显示
    p <- p + theme(axis.text.x=element_blank()) # 删除x轴刻度
    p <- p + theme(panel.background = element_rect(colour="black")) # 添加框线
    p <- p + theme(plot.margin = unit(c(0,0,0,0), "cm")) # 尽量删除图外空白
    return(p)
}
f_VlnBoxPlot_1 <- function(scRNAo, feature_type, features, isLast, ...){
    feature = features[1]
    p <- f_VlnBoxPlot_0(scRNAo, feature, ...)
    if (length(features) > 1){
        for (feature in features[2:length(features)]){
            p <- p / f_VlnBoxPlot_0(scRNAo, feature, ...)
        }
    }
    if (isLast){
        p <- p + theme(axis.text.x=element_text(hjust=0.5,angle=45)) # 添加x轴坐标
    }
    p <- as.ggplot(p) + facet_set(feature_type, side = "r") # 添加基因集说明
    p <- p + theme(strip.text.y = element_text(size = 15)) # 设置说明文字大小
    return(p)
}
f_VlnBoxPlot_2 <- function(markers){
    markers <- markers[order(markers$celltype),]
    feature_types  <- unique(markers$celltype)
    res <- list()
    for (feature_type in feature_types){
        res[[feature_type]] = NULL
    }
    for (lc_i in 1:nrow(markers)){
        feature_type <- markers[lc_i, 'celltype']
        feature <- markers[lc_i, 'marker']
        res[[feature_type]]  <- c(res[[feature_type]], feature)
    }
    return(res)
}
pushR = function(foo, bar){
  foo[[length(foo)+1]] <- bar
  foo
}
popR = function(foo){
  foo[[length(foo)]]
}
f_VlnBoxPlot <- function(scRNAo, lc_markers, ...){
    markers <- f_VlnBoxPlot_2(lc_markers)
    feature_types  <- names(markers)
    last_feature_type <- feature_types[length(feature_types)]
    feature_type <- feature_types[1]
    p <- f_VlnBoxPlot_1(scRNAo, feature_type, markers[[feature_type]], feature_type==last_feature_type)
#     plist <- list()
#     plist <- pushR(plist, p)
#     p1 <- p # 作为参考标准图
    if (length(feature_types) > 1){
        for (feature_type in feature_types[2:length(feature_types)]){
            p2 <- f_VlnBoxPlot_1(scRNAo, feature_type, markers[[feature_type]], feature_type==last_feature_type)
#             plist <- pushR(plist, p2)
#             p2 <- p2 + xlim2(p1) # x轴与p1对齐
            p <- p / p2
        }
    }
#     p <- as.ggplot(p + theme(axis.text.x=element_text(hjust=0.5,angle=45))) # 添加x轴坐标
#     p <- plot_grid(plotlist = plist, ncol = 1, align = "v")
    return(p)
}
markers <- read.csv(header = T, text = '
celltype,marker
Naive CD4+ T,IL7R
Naive CD4+ T,CCR7
CD14+ Mono,CD14
CD14+ Mono,LYZ
Memory CD4+,S100A4
B,MS4A1
CD8+ T,CD8A
FCGR3A+ Mono,FCGR3A
FCGR3A+ Mono,MS4A7
NK,GNLY
NK,NKG7
DC,FCER1A
DC,CST3
Platelet,PPBP
')
c <- f_VlnBoxPlot(pbmc3k, markers)
c
```

## 第三步 尝试分面

```r
f_VlnBoxPlot_18 <- function(features){
    tb <- data.frame(table(features))
    tb <- tb[tb$Freq > 1,]
    return(tb)
}
 
f_VlnBoxPlot_19 <- function(tmp, features, groupN){
    dupl <- f_VlnBoxPlot_18(features)
    if(nrow(dupl) < 1){
        return(tmp)
    }
    for(lc_i in 1:nrow(dupl)){
        dupfeature <- dupl[lc_i, 'features']
        idx <- which(features == dupfeature)
        for (lc_j in 2:dupl[lc_i, 'Freq']){
            newfeature <- paste0(dupfeature,'.',lc_j)
#             features[[idx[lc_j]]] <- newfeature
#             tmp[[newfeature]] <- tmp[[dupfeature]]
            colnames(tmp)[[idx[lc_j]]] <- newfeature
        }
    }
#     tmp <- tmp[,c(features, groupN)]
    return(tmp)
}
 
f_VlnBoxPlot_20 <- function(scRNAo, features, groupN){
    tmp <- log1p(scRNAo[['RNA']]@counts)
#     tmp <- scRNAo[['RNA']]@counts
    tmp <- as.data.frame(t(as.matrix(tmp[features,])))
    types <- scRNAo[[groupN]]
    types[[groupN]] <- factor(types[[groupN]]) # 确保是因子
    newLevels <- levels(types[[groupN]])
    newLevels <- newLevels[order(newLevels)]
    types[[groupN]] <- factor(types[[groupN]], levels=newLevels) # 排序
    tmp <- cbind(tmp, types)
    tmp <- f_VlnBoxPlot_19(tmp, features, groupN)
    return(tmp)
}
 
f_setRowName <- function(lc_df, lc_colName){
    lc_df <- lc_df[order(lc_df[[lc_colName]]),]
    tp_index <- duplicated(lc_df[[lc_colName]])
    lc_df <- lc_df[!tp_index,]
    rownames(lc_df) <- lc_df[[lc_colName]]
    lc_df
}
 
f_VlnBoxPlot_25 <- function(markers){
    dupl <- f_VlnBoxPlot_18(markers$marker)
    if(nrow(dupl) >= 1){
        for(lc_i in 1:nrow(dupl)){
            dupfeature <- dupl[lc_i, 'features']
            idx <- which(markers$marker == dupfeature)
            for (lc_j in 2:dupl[lc_i, 'Freq']){
                newfeature <- paste0(dupfeature,'.',lc_j)
                markers[idx[lc_j], 'marker'] <- newfeature
            }
        }
    }
    markers <- f_setRowName(markers, 'marker') # 准备marker到类型的映射
    markers[['celltype']] <- factor(markers[['celltype']], ordered = T)
    col <- f_VlnBoxPlot_col(length(levels(markers[['celltype']])))
    markers[['colour']] <- col[as.numeric(markers[['celltype']])]
    return(markers)
}
 
require(reshape2)
f_VlnBoxPlot_21 <- function(scRNAo, markers, groupN){
    data <- f_VlnBoxPlot_20(scRNAo, markers[['marker']], groupN) # 获取宽数据
    data <- melt(data, id.vars = groupN) # 宽数据变长数据
    
    markers <- f_VlnBoxPlot_25(markers) # 准备marker到颜色的映射
    data[['facet_fill_color']]  <- markers[as.character(data[['variable']]), 'colour'] # 进行映射
    newLevels <- data[['facet_fill_color']][!duplicated(data[['facet_fill_color']])] # 确保顺序
    data[['facet_fill_color']] <- factor(data[['facet_fill_color']], levels=newLevels)
    
    return(data)
}
 
f_VlnBoxPlot_23 <- function(scRNAo, markers, groupN){
    markers <- markers[order(markers$celltype),] # 确保顺序正确
    theme_s <- VlnPlot(scRNAo, features = markers[1,'marker'], slot = "counts", log = TRUE, group.by = groupN) # 借用 Seurat 的主题
    data <- f_VlnBoxPlot_21(scRNAo, markers, groupN) # 准备好数据
    p <- ggplot(data, aes(x=!!sym(groupN), y=value, fill= !!sym(groupN)))
    p <- p + theme_s$theme + NoLegend()  # 借用 Seurat 的主题
    p <- p + geom_violin() # 绘制小提琴图
    p <- p + stat_ydensity(trim = TRUE, scale = 'width', adjust = 1) # 绘制小提琴图
    p <- p + geom_boxplot(width=0.618) # 绘制箱型图
    p <- p + stat_summary(fun="mean",geom="point",color='white') # 添加均值点
    p <- p + theme(plot.title=element_blank()) # 删除总标题
    p <- p + labs(x=NULL, y=NULL) # 删除xy轴标题
    p <- p + scale_y_continuous('', breaks = floor(max(layer_scales(p)$y$range$range))) # 美化y轴刻度
#     p <- p + theme(panel.background = element_rect(colour="black")) # 添加框线
    p <- p + theme(plot.margin = unit(c(0,0,0,0), "cm")) # 尽量删除图外空白
    p <- p + facet_grid(variable ~ .) # 添加分页
    nx <- length(unlist(unique(scRNAo[[groupN]])))
    p <- p + scale_fill_manual(values=f_VlnBoxPlot_col(nx))
    x_r <- layer_scales(p)$x$range$range # 保存x轴刻度
    p <- p + theme(axis.text.x=element_blank()) # 删除x轴刻度
    p <- f_VlnBoxPlot_26(data, p)
#     layer_scales(p)$x$range$range <- x_r 
    nr <- length(x_r) + 2
    p <- p + scale_x_continuous( # 恢复x轴刻度
        limits = c(1/nr/2,1-1/nr/2),
        breaks = seq(0,1,length.out=nr)[2:(nr-1)],
        label = x_r
    )
    p <- p + theme(axis.text.x=element_text(hjust = 0.5, angle = 45))
    return(p)
}
 
require(treeio)
require(ggplot2)
require(ggtree)
require(tidytree)
# require(aplot)
f_VlnBoxPlot_24 <- function(markers){
    markers <- markers[order(markers$celltype),]
#     pstandard <- ggplot(markers, aes(y = marker)) # 创建标准y轴顺序
    feature_types  <- unique(markers$celltype) # 创建根节点
    feature_types <- as.data.frame(feature_types)
    feature_types[['root']] = ' '
    feature_types <- feature_types[, c('root', 'feature_types')]
    colnames(feature_types) <- colnames(markers)
    
    y = as.phylo(rbind(feature_types, markers)) # 转换成树
    
    yy = as_tibble(y) %>%  # 统一叶子节点的颜色
    mutate(cat = ifelse(node %in% parent, 1, parent))
    yy$cat[rootnode(y)] = 0
    
    p <- ggtree(as.treedata(yy), ladderize=F, layout='roundrect') +  # 通过 ggtree 绘图
    geom_nodelab(aes(x=x*0.5, label=label, 
                   fill=factor(cat)), hjust=0.5, geom='text', angle=90) + 
    geom_tiplab(aes(label=label, fill=factor(cat)), geom='text', angle=45, hjust=0.5) +
    scale_y_reverse() +
    hexpand(.2) + hexpand(.06, -1) + 
    theme(legend.position = 'none')
#     p <- p + ylim2(pstandard) + scale_y_reverse() 
    p <- p + theme(plot.margin = unit(c(0,0,0,0), "cm")) # 尽量删除图外空白
    return(p)
}
 
require(ggplotify)
require(gtable)
require(grid)
gtable_select <- function (x, ...)
{
    matches <- c(...)
    x$layout <- x$layout[matches, , drop = FALSE]
    x$grobs <- x$grobs[matches]
    x
}
gtable_stack <- function(g1, g2){
    g1$grobs <- c(g1$grobs, g2$grobs)
    g1$layout <- rbind(g1$layout, g2$layout)
    g1
}
f_VlnBoxPlot_26 <- function(dat, p){
    p <- p + theme(strip.background=element_blank())  # 去掉背景
    dummy <- p # 魔改出facet的填充
    dummy$layers <- NULL
    dummy <- dummy + geom_rect(data=dat, xmin=-Inf, ymin=-Inf, xmax=Inf, ymax=Inf,
                           aes(fill = facet_fill_color))
#     print(dummy)
    g1 <- ggplotGrob(p)
    g2 <- ggplotGrob(dummy)
    panels <- grepl(pattern="panel", g2$layout$name)
    strips <- grepl(pattern="strip-right", g2$layout$name)
    g2$grobs[strips] <- replicate(sum(strips), nullGrob(), simplify = FALSE)
    g2$layout$l[panels] <- g2$layout$l[panels] + 1
    g2$layout$r[panels] <- g2$layout$r[panels] + 2
    new_strips <- gtable_select(g2, panels  strips)
#     print(as.ggplot(new_strips))
    new_plot <- gtable_stack(g1, new_strips)
    p <- as.ggplot(new_plot)
    return(p)
}
 
require(RColorBrewer)
col_Paired <- colorRampPalette(brewer.pal(12, "Paired"))
f_VlnBoxPlot_col <- function(nx){
    if (nx <= 12){
        return(brewer.pal(nx, "Paired"))
    }else{
        return(col_Paired(nx))
    }
}
 
f_VlnBoxPlot <- function(scRNAo, markers, groupN){
    p2 <- f_VlnBoxPlot_24(markers)
    p <- f_VlnBoxPlot_23(scRNAo, markers , groupN)
    p2 + p + plot_layout(ncol = 2, widths = c(1, 5))  
}

f_VlnBoxPlot_simple <- function(scRNAo, markers, groupN){
    markers <- markers[order(markers$celltype),] # 确保顺序正确
    theme_s <- VlnPlot(scRNAo, features = markers[1,'marker'], slot = "counts", log = TRUE, group.by = groupN) # 借用 Seurat 的主题
    data <- f_VlnBoxPlot_21(scRNAo, markers, groupN) # 准备好数据
    p <- ggplot(data, aes(x=!!sym(groupN), y=value, fill= !!sym(groupN)))
    p <- p + theme_s$theme + NoLegend()  # 借用 Seurat 的主题
    p <- p + geom_violin() # 绘制小提琴图
    p <- p + stat_ydensity(trim = TRUE, scale = 'width', adjust = 1) # 绘制小提琴图
    p <- p + geom_boxplot(width=0.618) # 绘制箱型图
    p <- p + stat_summary(fun="mean",geom="point",color='white') # 添加均值点
    p <- p + theme(plot.title=element_blank()) # 删除总标题
    p <- p + labs(x=NULL, y=NULL) # 删除xy轴标题
    p <- p + scale_y_continuous('', breaks = floor(max(layer_scales(p)$y$range$range))) # 美化y轴刻度
#     p <- p + theme(panel.background = element_rect(colour="black")) # 添加框线
    p <- p + theme(plot.margin = unit(c(0,0,0,0), "cm")) # 尽量删除图外空白
    p <- p + facet_grid(variable ~ .) # 添加分页
    nx <- length(unlist(unique(scRNAo[[groupN]])))
    p <- p + scale_fill_manual(values=f_VlnBoxPlot_col(nx))
    p <- f_VlnBoxPlot_26(data, p)
    return(p)
}
```

## 第四步 最终效果展示

```r
markers <- read.csv(header = T, text = '
celltype,marker
Naive CD4+ T,IL7R
Naive CD4+ T,CCR7
CD14+ Mono,CD14
CD14+ Mono,LYZ
Memory CD4+,IL7R
Memory CD4+,S100A4
B,MS4A1
CD8+ T,CD8A
FCGR3A+ Mono,FCGR3A
FCGR3A+ Mono,MS4A7
NK,GNLY
NK,NKG7
DC,FCER1A
DC,CST3
Platelet,PPBP
')

f_VlnBoxPlot(pbmc3k, markers , 'seurat_annotations')
```

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/VlnBox_5.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/VlnBox_5.webp)