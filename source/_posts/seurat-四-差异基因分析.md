---
title: Seurat (四) 差异基因分析
tags: []
id: '873'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-10-03 20:09:58
---

## 第一步 火山图

```
Idents(sc_Neuron) <- sc_Neuron[['hmca_class']]
all_markers <- FindAllMarkers(sc_Neuron, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)

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
    df$logFC[df$logFC > 10] = 10
    df$logFC[df$logFC < -10] = -10
    res <- ggplot() + geom_point(aes(logFC, `-log10(P)`, col=I(col)), data = df) 
    res <- res + theme_bw() + theme(panel.grid=element_line(colour=NA))
    res <- res + geom_hline(yintercept=-log10(Threshold_p), linetype="longdash")
    res <- res + geom_vline(xintercept=c(Threshold_logFC, -Threshold_logFC), linetype="longdash")
    res <- res + geom_text_repel(data=df[lc_idx,],aes(logFC,`-log10(P)`,label=gene), force=T, max.overlaps=Inf)
    res
}

```

## 第二步 热图

```
f_cluster_averages <- function(lc_scRNA, lc_metaN='ident'){
    # 切分出Clusters
    lc_clusters <- SplitObject(lc_scRNA, split.by = lc_metaN)
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- lc_clusters[[lc_i]][[lc_clusters[[lc_i]]@active.assay]]@scale.data
    }
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- apply(lc_clusters[[lc_i]],1,mean)
    }
    lc_clusters <- data.frame(lc_clusters)
    scale(lc_clusters)
}

library(clusterProfiler)
library(pheatmap)
library(ggdendro)
f_DEG_hclust <- function(lc_counts){
    ggdendrogram(hclust(dist(t(lc_counts))), rotate = T, size = 3)+theme(axis.text = element_text(size=14,face = "bold"))
}

tp_d <- f_cluster_averages(sc_Neuron, "orig.ident")
f_DEG_hclust(tp_d)

f_DEG_pheatmap_choose_matrix <- function(lc_tp_d, lc_significant_markers, lc_n = 120){
    res <- subset(lc_significant_markers, abs(avg_log2FC) > 1)
    res <- res[order(abs(res$avg_log2FC), decreasing = T),]
    res <- head(unique(res$gene), n = lc_n)
    res <- lc_tp_d[res,]
    res
}

Idents(sc_Neuron) <- sc_Neuron[['orig.ident']]
all_markers <- FindAllMarkers(sc_Neuron, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)

f_DEG_Volcano(all_markers$avg_log2FC, all_markers$p_val_adj, all_markers$gene)

```

## 第三步 一些奇怪的图

```
f_br_cluster <- function(sObject, lc_groupN, lc_labelN, lc_prop = F){
    lc_all <- unique(sObject[[lc_labelN]])
    rownames(lc_all) <- lc_all[[1]]
    colnames(lc_all) <- "CB"
    lc_tp  <- SplitObject(sObject, split.by = lc_groupN)
    for(lc_i in 1:length(lc_tp)){
        if(lc_prop){
            lc_tp[[lc_i]] <- prop.table(table(lc_tp[[lc_i]][[lc_labelN]]))
        }else{
            lc_tp[[lc_i]] <- table(lc_tp[[lc_i]][[lc_labelN]])
        }
    }
    for(lc_name in names(lc_tp)){
        lc_all[[lc_name]] = 0
        lc_all[names(lc_tp[[lc_name]]), lc_name] = lc_tp[[lc_name]]
    }
    lc_all[,-1]
}

f_q2l <- function(lc_q){
    res  <- NULL
    for(lc_c in colnames(lc_q)){
        for(lc_r in rownames(lc_q)){
            res  <- rbind(res, c(group=lc_c, label=lc_r, value=lc_q[lc_r,lc_c]))
        }
    }
    res <- data.frame(res)
    res$value = as.numeric(res$value)
    res
}

library(RColorBrewer)
library(ggplot2)
col_Paired <- colorRampPalette(brewer.pal(12, "Paired"))
f_q_frequnency <- function(lc_q){
    ggplot(f_q2l(lc_q),mapping = aes(group,value,fill=label))+
    geom_bar(stat='identity',position='fill') + scale_fill_manual(values= col_Paired(nrow(lc_q)))+
    labs(x = 'group',y = 'frequnency') +
    theme(axis.title =element_text(size = 16),axis.text =element_text(size = 14, color = 'black'))+
    theme(axis.text.x = element_text(angle = 45, hjust = 1))+coord_flip() 
}

options(repr.plot.width=9, repr.plot.height=5)
tp_test <- f_br_cluster(sc_Neuron, 'orig.ident', 'hmca_class')
f_q_frequnency(tp_test)
friedman.test(as.matrix(tp_test))
chisq.test(as.matrix(tp_test), simulate.p.value = TRUE)
fisher.test(as.matrix(tp_test), simulate.p.value = TRUE)

f_DoHeatmap <- function(sObject, lc_significant_markers, lc_groupN){
    lc_significant_markers %>%
    group_by(cluster) %>%
    top_n(n = 10, wt = abs(avg_log2FC)) -> top10
    DoHeatmap(sc_Neuron, group.by = lc_groupN, features = top10$gene) + scale_fill_gradientn(colors = c('#440154', '#25848D', "yellow"))
}

```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/qf.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/qf.webp)