---
title: cellchat (四) 细胞间通讯分析
tags: []
id: '1614'
categories:
  - - 单细胞下游分析
date: 2022-02-12 18:54:49
---

## 信息流分析

```r
f_CC_IO_iN2pN <- function(cellchat, iN){
    iN <- gsub('−','-', iN)
    unique(cellchat@DB$interaction[cellchat@DB$interaction$interaction_name_2 %in% iN, 'pathway_name'])
}
require(ComplexHeatmap)
f_CC_cmp_IO_signaling  <- function(object.list, pattern='all', color.heatmap = "OrRd", width = 5, height = 6, s_pathway=NULL){
    i = 1
    pathway.union <- union(object.list[[i]]@netP$pathways, object.list[[i+1]]@netP$pathways)
    if(!is.null(s_pathway)){pathway.union <- intersect(pathway.union, s_pathway)}
    ht1 = netAnalysis_signalingRole_heatmap(object.list[[i]], pattern = pattern, signaling = pathway.union, title = names(object.list)[i], width = width, height = height, color.heatmap = color.heatmap)
    ht2 = netAnalysis_signalingRole_heatmap(object.list[[i+1]], pattern = pattern, signaling = pathway.union, title = names(object.list)[i+1], width = width, height = height, color.heatmap = color.heatmap)
    draw(ht1 + ht2, ht_gap = unit(0.5, "cm"))
}
```

```r
SS_s_p <- unlist(openxlsx::read.xlsx(xlsxFile = "SS_selected_signaling_fibroblasts.xlsx", sheet = 1, colNames = F))
ER_s_p <- unlist(openxlsx::read.xlsx(xlsxFile = "ER_selected_signaling_fibroblasts.xlsx", sheet = 1, colNames = F))
CC_s_p <- unlist(openxlsx::read.xlsx(xlsxFile = "CC_selected_signaling_fibroblasts.xlsx", sheet = 1, colNames = F))

pdf(file = 'IO_signaling_all.pdf', height = 8, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_IO_signaling(SS_l, width = 10, height = 12, s_pathway = f_CC_IO_iN2pN(SS_H, SS_s_p))
f_CC_cmp_IO_signaling(ER_l, width = 10, height = 4, s_pathway = f_CC_IO_iN2pN(ER_H, ER_s_p))
f_CC_cmp_IO_signaling(CC_l, width = 10, height = 4, s_pathway = f_CC_IO_iN2pN(CC_H, CC_s_p))
dev.off()

pdf(file = 'IO_signaling_outgoing.pdf', height = 8, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_IO_signaling(SS_l, pattern = 'outgoing', color.heatmap = 'BuGn', width = 10, height = 12, s_pathway = f_CC_IO_iN2pN(SS_H, SS_s_p))
f_CC_cmp_IO_signaling(ER_l, pattern = 'outgoing', color.heatmap = 'BuGn', width = 10, height = 4, s_pathway = f_CC_IO_iN2pN(ER_H, ER_s_p))
f_CC_cmp_IO_signaling(CC_l, pattern = 'outgoing', color.heatmap = 'BuGn', width = 10, height = 4, s_pathway = f_CC_IO_iN2pN(CC_H, CC_s_p))
dev.off()

pdf(file = 'IO_signaling_incoming.pdf', height = 8, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_IO_signaling(SS_l, pattern = 'incoming', color.heatmap = 'GnBu', width = 10, height = 12, s_pathway = f_CC_IO_iN2pN(SS_H, SS_s_p))
f_CC_cmp_IO_signaling(ER_l, pattern = 'incoming', color.heatmap = 'GnBu', width = 10, height = 4, s_pathway = f_CC_IO_iN2pN(ER_H, ER_s_p))
f_CC_cmp_IO_signaling(CC_l, pattern = 'incoming', color.heatmap = 'GnBu', width = 10, height = 4, s_pathway = f_CC_IO_iN2pN(CC_H, CC_s_p))
dev.off()
```

## 功能失调信号

```r
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
```

```r
dSS <- f_CC_cmp_dysfunctional_signaling(SS)
dER <- f_CC_cmp_dysfunctional_signaling(ER)
dCC <- f_CC_cmp_dysfunctional_signaling(CC)
```

```r
f_CC_s_p_intersect <- function(net_info, DB_s_p){
    DB_s_p <- gsub('−','-', DB_s_p)
    DB_s_p <- DB_s_p[DB_s_p %in% net_info$interaction_name_2]
    net_info <- net_info[net_info$interaction_name_2 %in% DB_s_p, c("interaction_name", "interaction_name_2"), drop = F]
    net_info <- net_info[, "interaction_name", drop = F]
    net_info
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
    }else{
        pairLR.use.up <- f_CC_s_p_intersect(f_res$net_up, pairLR.use.up)
    }
    gg1 <- netVisual_bubble(cellchat, pairLR.use = pairLR.use.up, sources.use = sources.use, targets.use = targets.use, comparison = c(1, 2),  angle.x = 90, remove.isolate = T,title.name = paste0("Up-regulated signaling in ", pos.dataset))
    #> Comparing communications on a merged object
    if(is.null(pairLR.use.down)){
        pairLR.use.down = f_res$net_down[, "interaction_name", drop = F]
    }else{
        pairLR.use.down <- f_CC_s_p_intersect(f_res$net_down, pairLR.use.down)
    }
    gg2 <- netVisual_bubble(cellchat, pairLR.use = pairLR.use.down, sources.use = sources.use, targets.use = targets.use, comparison = c(1, 2),  angle.x = 90, remove.isolate = T,title.name = paste0("Down-regulated signaling in ", pos.dataset))
    #> Comparing communications on a merged object
    gg1 + gg2
}
```

```r
flt <- c('Fibroblasts', 'TAM', 'Mast', 'S100A8+ monocytes', 'Proliferating T', 'Proliferating Myeloid', 'Luminal', 'Endothelial', 'Basal cell', 'cDC2')

pdf(file = 'dysfunctional_signaling.pdf', height = 12, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_dysfunctional_signaling_draw(SS, f_res = dSS, sources.use = c('Fibroblasts'), targets.use = flt, pairLR.use.up = SS_s_p, pairLR.use.down = SS_s_p)
f_CC_cmp_dysfunctional_signaling_draw(ER, f_res = dER, sources.use = c('Fibroblasts'), targets.use = flt, pairLR.use.up = ER_s_p, pairLR.use.down = ER_s_p)
f_CC_cmp_dysfunctional_signaling_draw(CC, f_res = dCC, sources.use = c('Fibroblasts'), targets.use = flt, pairLR.use.up = CC_s_p, pairLR.use.down = CC_s_p)
dev.off()

pdf(file = 'dysfunctional_signaling_t.pdf', height = 12, width = 12)
par(mfrow = c(3,1), xpd=TRUE)
f_CC_cmp_dysfunctional_signaling_draw(SS, f_res = dSS, targets.use = c('Fibroblasts'), sources.use = flt, pairLR.use.up = SS_s_p, pairLR.use.down = SS_s_p)
f_CC_cmp_dysfunctional_signaling_draw(ER, f_res = dER, targets.use = c('Fibroblasts'), sources.use = flt, pairLR.use.up = ER_s_p, pairLR.use.down = ER_s_p)
f_CC_cmp_dysfunctional_signaling_draw(CC, f_res = dCC, targets.use = c('Fibroblasts'), sources.use = flt, pairLR.use.up = CC_s_p, pairLR.use.down = CC_s_p)
dev.off()
```

## 细胞通路弦图

```r
f_CC_IO_iN2pN_2 <- function(cellchat, iN){
    iN <- gsub('−','-', iN)
    unique(cellchat@DB$interaction[cellchat@DB$interaction$interaction_name_2 %in% iN, c('pathway_name', 'interaction_name_2')]['pathway_name'])
}
f_CC_cmp_nV_c_p  <- function(object.list, sources.use=NULL, targets.use=NULL, s_pathway=NULL, title.name){
    for (i in 1:length(object.list)) {
        if(!is.null(s_pathway)){
            lc_pairLR.use <- f_CC_IO_iN2pN_2(object.list[[i]], s_pathway)
        }else{lc_pairLR.use = s_pathway}
        try(netVisual_chord_gene(object.list[[i]], sources.use = sources.use, pairLR.use = lc_pairLR.use, targets.use = targets.use, slot.name = "netP", title.name = paste0("Signaling pathways - ", names(object.list)[i]), legend.pos.x = 10))
    }
}
```

```r
pdf(file = 'nV_c_p.pdf', height = 12, width = 12)
f_CC_cmp_nV_c_p(SS_l, sources.use = c('Fibroblasts'), targets.use = flt, s_pathway = SS_s_p)
f_CC_cmp_nV_c_p(ER_l, sources.use = c('Fibroblasts'), targets.use = flt, s_pathway = ER_s_p)
f_CC_cmp_nV_c_p(CC_l, sources.use = c('Fibroblasts'), targets.use = flt, s_pathway = CC_s_p)
dev.off()

pdf(file = 'nV_c_p_t.pdf', height = 12, width = 12)
f_CC_cmp_nV_c_p(SS_l, targets.use = c('Fibroblasts'), sources.use = flt, s_pathway = SS_s_p)
f_CC_cmp_nV_c_p(ER_l, targets.use = c('Fibroblasts'), sources.use = flt, s_pathway = ER_s_p)
f_CC_cmp_nV_c_p(CC_l, targets.use = c('Fibroblasts'), sources.use = flt, s_pathway = CC_s_p)
dev.off()
```