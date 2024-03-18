---
title: GEO (一) 绘图总结
tags: []
id: '861'
categories:
  - - 生物信息学
  - - 组织测序分析
date: 2021-10-03 17:04:43
---

## 第一步 火山图

```
library(ggplot2)
library(ggrepel)
f_DEG_Volcano <- function(lc_logFC, lc_p, lc_gene, Threshold_logFC = 1, Threshold_p = 0.05, lc_rep=1:10){
    col_vector = rep(rgb(108, 200, 228, maxColorValue = 255), length(lc_logFC))
    col_vector[lc_p < Threshold_p & lc_logFC > Threshold_logFC] = rgb(226, 61, 75, maxColorValue = 255)
    col_vector[lc_p < Threshold_p & lc_logFC < -Threshold_logFC] = rgb(232, 168, 71, maxColorValue = 255)
    lc_p[lc_p < 1e-10] = 1e-10
    lc_p[lc_p > 1  is.na(lc_p)] = 1
    df = data.frame(logFC <- lc_logFC, `-log10(P)` <- -log10(lc_p), col <- col_vector, gene <- lc_gene)
    colnames(df) <- c('logFC', '-log10(P)', "col", "gene")
    lc_tp_logFC <- df$logFC
    lc_tp_logFC[lc_p>=Threshold_p] = 0
    lc_idx <- order(lc_tp_logFC)[c(lc_rep, length(lc_gene)+1-lc_rep)]
    res <- ggplot() + geom_point(aes(logFC, `-log10(P)`, col=I(col)), data = df) 
    res <- res + theme_bw() + theme(panel.grid=element_line(colour=NA))
    res <- res + geom_hline(yintercept=-log10(Threshold_p), linetype="longdash")
    res <- res + geom_vline(xintercept=c(Threshold_logFC, -Threshold_logFC), linetype="longdash")
    res <- res + geom_text_repel(data=df[lc_idx,],aes(logFC,`-log10(P)`,label=gene), force=T, max.overlaps=Inf)
    res
}

```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/v-1.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/v-1.webp)

## 第二步 箱型图

```
library(ggplot2)
library(reshape2)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)

f_gl_boxp <- function(lc_exp, lc_g, lc_delta_ylim){
    lc_exp_L = melt(lc_exp)
    colnames(lc_exp_L) = c('symbol','sample','value')
    lc_exp_L$group = lc_g
    p=ggplot(lc_exp_L,aes(x=sample,y=value,fill=group))+geom_boxplot(outlier.shape = NA)
    p=p+theme_set(theme_set(theme_bw(base_size=20)))
    p=p+theme(text=element_text(face='bold'),axis.text.x=element_text(angle=30,hjust=1),axis.title=element_blank())
    lc_ylim = unlist(by(lc_exp_L,
     lc_exp_L$sample,
     function(x){boxplot.stats(x$value)$stats[c(1, 5)]}))
    lc_ylim <- c(min(lc_ylim), max(lc_ylim)) + lc_delta_ylim
    p=p + coord_cartesian(ylim = lc_ylim)
    p
}

f_icg_boxp <- function(lc_exp, lc_icg, lc_g){
    lc_exp_L = melt(lc_exp[lc_icg,])
    lc_exp_L <- cbind(lc_exp_L, rownames(lc_exp_L))
    colnames(lc_exp_L)=c('value','sample')
    lc_exp_L$group = lc_g
    p=ggplot(lc_exp_L,aes(x=group,y=value,fill=group))+geom_boxplot()
    p=p+stat_summary(fun="mean",geom="point",shape=23,size=3,fill="red")
    p=p+theme_set(theme_set(theme_bw(base_size=20)))
    p=p+theme(text=element_text(face='bold'),axis.text.x=element_text(angle=30,hjust=1),axis.title=element_blank())
    p=p+ggtitle(lc_icg)+theme(plot.title = element_text(hjust = 0.5))
    p
}

library("illuminaMousev2.db")
f_id2name <- function(lc_exp, lc_db){
    lc_ids=toTable(lc_db)
    res <- lc_exp[rownames(lc_exp) %in% lc_ids[[1]],]
    lc_ids=lc_ids[match(rownames(res),lc_ids[[1]]),]
    lc_tmp = by(res,
         lc_ids[[2]],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res = res[rownames(res) %in% lc_probes,]
    rownames(res)=lc_ids[match(rownames(res),lc_ids[[1]]),2]
    res
}

# icg
tp_counts <- f_id2name(gl_counts, illuminaMousev2SYMBOL)

```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/box-2.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/box-2.webp)

## 第三步 热图

```
f_DEG_pheatmap <- function(choose_matrix){
    choose_matrix = t(scale(t(choose_matrix)))
    pheatmap(choose_matrix)
}

choose_gene = rownames(tT2[tT2[["P.Value"]] < 0.05 & abs(tT2$logFC) > 1,])
choose_matrix = gl_counts[choose_gene,]
choose_matrix = f_id2name(choose_matrix, illuminaMousev2SYMBOL)
options(repr.plot.width=9, repr.plot.height=26)

```