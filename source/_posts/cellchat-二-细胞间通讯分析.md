---
title: cellchat (二) 细胞间通讯分析
tags: []
id: '1579'
categories:
  - - 单细胞下游分析
date: 2022-02-11 02:25:27
---

## 前置文章

[cellchat (一) 细胞间通讯分析](https://limour.top/1070.html)

## 第一步 合并数据

```r
library(CellChat)
library(patchwork)

#01
SS_H = readRDS(file = 'SS_H.rds')
SS_C = readRDS(file = 'SS_C.rds')

#02
ER_H = readRDS(file = 'ER_H.rds')
ER_C = readRDS(file = 'ER_C.rds')

#03
CC_H = readRDS(file = 'CC_H.rds')
CC_C = readRDS(file = 'CC_C.rds')

SS_l <- list(HSPC = SS_H, CRPC = SS_C)
ER_l <- list(HSPC = ER_H, CRPC = ER_C)
CC_l <- list(HSPC = CC_H, CRPC = CC_C)

SS <- mergeCellChat(SS_l, add.names = names(SS_l))
ER <- mergeCellChat(ER_l, add.names = names(ER_l))
CC <- mergeCellChat(CC_l, add.names = names(CC_l))
```

## 第二步 定义绘图函数

```r
f_CC_cmp_nVheatmap <- function(cellchat){
    gg1 <- netVisual_heatmap(cellchat)
    #> Do heatmap based on a merged object
    gg2 <- netVisual_heatmap(cellchat, measure = "weight")
    #> Do heatmap based on a merged object
    gg1 + gg2
}

f_CC_cmp_major_sources_and_targets  <- function(object.list){
    num.link <- sapply(object.list, function(x) {rowSums(x@net$count) + colSums(x@net$count)-diag(x@net$count)})
    weight.MinMax <- c(min(num.link), max(num.link)) # control the dot size in the different datasets
    gg <- list()
    for (i in 1:length(object.list)) {
      gg[[i]] <- netAnalysis_signalingRole_scatter(object.list[[i]], title = names(object.list)[i], weight.MinMax = weight.MinMax)
    }
    #> Signaling role analysis on the aggregated cell-cell communication network from all signaling pathways
    #> Signaling role analysis on the aggregated cell-cell communication network from all signaling pathways
    print(patchwork::wrap_plots(plots = gg))
}

f_CC_cmp_SG_F <- function(cellchat){
    cellchat <- computeNetSimilarityPairwise(cellchat, type = "functional")
    #> Compute signaling network similarity for datasets 1 2
    cellchat <- netEmbedding(cellchat, type = "functional")
    #> Manifold learning of the signaling networks for datasets 1 2
    cellchat <- netClustering(cellchat, type = "functional")
    #> Classification learning of the signaling networks for datasets 1 2
    # Visualization in 2D-space
    netVisual_embeddingPairwise(cellchat, type = "functional", label.size = 3.5)
    #> 2D visualization of signaling networks from datasets 1 2
}

f_CC_cmp_SG_S <- function(cellchat){
    cellchat <- computeNetSimilarityPairwise(cellchat, type = "structural")
    #> Compute signaling network similarity for datasets 1 2
    cellchat <- netEmbedding(cellchat, type = "structural")
    #> Manifold learning of the signaling networks for datasets 1 2
    cellchat <- netClustering(cellchat, type = "structural")
    #> Classification learning of the signaling networks for datasets 1 2
    # Visualization in 2D-space
    netVisual_embeddingPairwise(cellchat, type = "structural", label.size = 3.5)
    #> 2D visualization of signaling networks from datasets 1 2
}

f_CC_cmp_overall_information_flow  <- function(cellchat){
    gg1 <- rankNet(cellchat, mode = "comparison", stacked = T, do.stat = TRUE)
    gg2 <- rankNet(cellchat, mode = "comparison", stacked = F, do.stat = TRUE)
    gg1 + gg2
}

f_CC_cmp_IO_signaling  <- function(object.list, pattern='all', color.heatmap = "OrRd", width = 5, height = 6){
    i = 1
    # combining all the identified signaling pathways from different datasets 
    pathway.union <- union(object.list[[i]]@netP$pathways, object.list[[i+1]]@netP$pathways)
    ht1 = netAnalysis_signalingRole_heatmap(object.list[[i]], pattern = pattern, signaling = pathway.union, title = names(object.list)[i], width = width, height = height, color.heatmap = color.heatmap)
    ht2 = netAnalysis_signalingRole_heatmap(object.list[[i+1]], pattern = pattern, signaling = pathway.union, title = names(object.list)[i+1], width = width, height = height, color.heatmap = color.heatmap)
    draw(ht1 + ht2, ht_gap = unit(0.5, "cm"))
}

f_CC_cmp_dysfunctional_signaling  <- function(cellchat, pos.dataset = 'CRPC', neg.dataset='HSPC'){
    # define a char name used for storing the results of differential expression analysis
    features.name = pos.dataset
    # perform differential expression analysis
    cellchat <- identifyOverExpressedGenes(cellchat, group.dataset = "datasets", pos.dataset = pos.dataset, features.name = features.name, only.pos = FALSE, thresh.pc = 0.1, thresh.fc = 0.1, thresh.p = 1)
    #> Use the joint cell labels from the merged CellChat object
    # map the results of differential expression analysis onto the inferred cell-cell communications to easily manage/subset the ligand-receptor pairs of interest
    net <- netMappingDEG(cellchat, features.name = features.name)
    # extract the ligand-receptor pairs with upregulated ligands in LS
    net.up <- subsetCommunication(cellchat, net = net, datasets = pos.dataset,ligand.logFC = 0.2, receptor.logFC = NULL)
    # extract the ligand-receptor pairs with upregulated ligands and upregulated recetptors in NL, i.e.,downregulated in LS
    net.down <- subsetCommunication(cellchat, net = net, datasets = neg.dataset,ligand.logFC = -0.1, receptor.logFC = -0.1)
    gene.up <- extractGeneSubsetFromPair(net.up, cellchat)
    gene.down <- extractGeneSubsetFromPair(net.down, cellchat)
    return(list(net_up=net.up, net_down=net.down, gene_up=gene.up, gene_down=gene.down))
}

f_CC_cmp_dysfunctional_signaling_draw <- function(cellchat, f_res=NULL, sources.use=NULL, targets.use=NULL, pos.dataset = 'CRPC', neg.dataset='HSPC', pairLR.use.up=NULL, pairLR.use.down=NULL){
    if(is.null(f_res)){
        f_res <- f_CC_cmp_dysfunctional_signaling(cellchat, pos.dataset = pos.dataset, neg.dataset = neg.dataset)
    }
    if(is.null(targets.use)){
        targets.use  <- as.character(unique(unlist(cellchat@meta$ident)))
    }
    if(is.null(sources.use)){
        sources.use  <- as.character(unique(unlist(cellchat@meta$ident)))
    }
    if(is.null(pairLR.use.up)){
        pairLR.use.up = f_res$net_up[, "interaction_name", drop = F]
    }
    gg1 <- netVisual_bubble(cellchat, pairLR.use = pairLR.use.up, sources.use = sources.use, targets.use = targets.use, comparison = c(1, 2),  angle.x = 90, remove.isolate = T,title.name = paste0("Up-regulated signaling in ", pos.dataset))
    #> Comparing communications on a merged object
    if(is.null(pairLR.use.down)){
        pairLR.use.down = f_res$net_down[, "interaction_name", drop = F]
    }
    gg2 <- netVisual_bubble(cellchat, pairLR.use = pairLR.use.down, sources.use = sources.use, targets.use = targets.use, comparison = c(1, 2),  angle.x = 90, remove.isolate = T,title.name = paste0("Down-regulated signaling in ", pos.dataset))
    #> Comparing communications on a merged object
    gg1 + gg2
}
```

## 第三步 绘图

```r
pdf(file = 'nVheatmap.pdf', height = 6, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_nVheatmap(SS)
f_CC_cmp_nVheatmap(ER)
f_CC_cmp_nVheatmap(CC)
dev.off()

pdf(file = 'major_sources_and_targets.pdf', height = 6, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_major_sources_and_targets(SS_l)
f_CC_cmp_major_sources_and_targets(ER_l)
f_CC_cmp_major_sources_and_targets(CC_l)
dev.off()

pdf(file = 'SG_F.pdf', height = 6, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_SG_F(SS)
f_CC_cmp_SG_F(ER)
f_CC_cmp_SG_F(CC)
dev.off()

pdf(file = 'SG_S.pdf', height = 6, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_SG_S(SS)
f_CC_cmp_SG_S(ER)
f_CC_cmp_SG_S(CC)
dev.off()

pdf(file = 'overall_information_flow.pdf', height = 6, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_overall_information_flow(SS)
f_CC_cmp_overall_information_flow(ER)
f_CC_cmp_overall_information_flow(CC)
dev.off()

pdf(file = 'IO_signaling_all.pdf', height = 12, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_IO_signaling(SS_l, width = 10, height = 24)
f_CC_cmp_IO_signaling(ER_l, width = 10, height = 24)
f_CC_cmp_IO_signaling(CC_l, width = 10, height = 24)
dev.off()

pdf(file = 'IO_signaling_outgoing.pdf', height = 12, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_IO_signaling(SS_l, pattern = 'outgoing', color.heatmap = 'BuGn', width = 10, height = 24)
f_CC_cmp_IO_signaling(ER_l, pattern = 'outgoing', color.heatmap = 'BuGn', width = 10, height = 24)
f_CC_cmp_IO_signaling(CC_l, pattern = 'outgoing', color.heatmap = 'BuGn', width = 10, height = 24)
dev.off()

pdf(file = 'IO_signaling_incoming.pdf', height = 12, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_IO_signaling(SS_l, pattern = 'incoming', color.heatmap = 'GnBu', width = 10, height = 24)
f_CC_cmp_IO_signaling(ER_l, pattern = 'incoming', color.heatmap = 'GnBu', width = 10, height = 24)
f_CC_cmp_IO_signaling(CC_l, pattern = 'incoming', color.heatmap = 'GnBu', width = 10, height = 24)
dev.off()

pdf(file = 'dysfunctional_signaling.pdf', height = 24, width = 24)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_dysfunctional_signaling_draw(SS, sources.use = c('Fibroblasts'))
f_CC_cmp_dysfunctional_signaling_draw(ER, sources.use = c('Fibroblasts'))
f_CC_cmp_dysfunctional_signaling_draw(CC, sources.use = c('Fibroblasts'))
dev.off()

pdf(file = 'dysfunctional_signaling_TAM.pdf', height = 24, width = 24)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_dysfunctional_signaling_draw(SS, sources.use = c('TAM'))
f_CC_cmp_dysfunctional_signaling_draw(ER, sources.use = c('TAM'))
f_CC_cmp_dysfunctional_signaling_draw(CC, sources.use = c('TAM'))
dev.off()

tss <- f_CC_cmp_dysfunctional_signaling(SS)
ter <- f_CC_cmp_dysfunctional_signaling(ER)
tcc <- f_CC_cmp_dysfunctional_signaling(CC)

pdf(file = 'dysfunctional_signaling_TAM_t.pdf', height = 24, width = 24)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_dysfunctional_signaling_draw(SS, f_res = tss, targets.use = c('TAM'))
f_CC_cmp_dysfunctional_signaling_draw(ER, f_res = ter, targets.use = c('TAM'))
f_CC_cmp_dysfunctional_signaling_draw(CC, f_res = tcc, targets.use = c('TAM'))
dev.off()

pdf(file = 'dysfunctional_signaling_t.pdf', height = 24, width = 24)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_dysfunctional_signaling_draw(SS, f_res = tss, targets.use = c('Fibroblasts'))
f_CC_cmp_dysfunctional_signaling_draw(ER, f_res = ter, targets.use = c('Fibroblasts'))
f_CC_cmp_dysfunctional_signaling_draw(CC, f_res = tcc, targets.use = c('Fibroblasts'))
dev.off()
```