---
title: "cellchat (三)\_和弦图"
tags: []
id: '1583'
categories:
  - - R绘图奇技淫巧
date: 2022-02-11 05:05:35
---

## 第一步 定义绘图函数

```r
f_CC_cmp_nV_df <- function(cellchat){
    g1 <- netVisual_diffInteraction(cellchat, weight.scale = T)
    g2 <- netVisual_diffInteraction(cellchat, weight.scale = T, measure = "weight")
}

f_CC_cmp_nV_chord_gene  <- function(object.list, cellchat, sources.use=NULL, targets.use=NULL, title.name=''){
    if(is.null(targets.use)){
        targets.use  <- as.character(unique(unlist(cellchat@meta$ident)))
    }
    if(is.null(sources.use)){
        sources.use  <- as.character(unique(unlist(cellchat@meta$ident)))
    }
    gg1 <- netVisual_chord_gene(object.list[[1]], sources.use = sources.use, targets.use = targets.use, lab.cex = 0.5, title.name = paste0(title.name, names(object.list)[1]))
    gg2 <- netVisual_chord_gene(object.list[[2]], sources.use = sources.use, targets.use = targets.use, lab.cex = 0.5, title.name = paste0(title.name, names(object.list)[2]))
}

f_CC_nV_split_s <- function(mat, sources.use){
    mat2 <- matrix(0, nrow = nrow(mat), ncol = ncol(mat), dimnames = dimnames(mat))
    idx <- order(mat[sources.use, ], decreasing = T)
    mat2 <- mat2[idx, idx]
    mat2[sources.use, colnames(mat2)] <- mat[sources.use, colnames(mat2)]
    min_mat <- min(mat2)
    if(min_mat<=0){
        mat2 <- mat2 - min_mat + 1e-20
    }
    g1 <- netVisual_circle(mat2, sources.use = sources.use, vertex.weight = mat2[sources.use, ], weight.scale = T, title.name = sources.use)
}
f_CC_nV_split_t <- function(mat, targets.use){
    mat2 <- matrix(0, nrow = nrow(mat), ncol = ncol(mat), dimnames = dimnames(mat))
    idx <- order(mat[, targets.use], decreasing = T)
    mat2 <- mat2[idx, idx]
    mat2[rownames(mat2), targets.use] <- mat[rownames(mat2), targets.use]
    min_mat <- min(mat2)
    if(min_mat<=0){
        mat2 <- mat2 - min_mat + 1e-20
    }
    g1 <- netVisual_circle(mat2, targets.use = targets.use, vertex.weight = mat2[, targets.use], weight.scale = T, title.name = targets.use) 
}
f_CC_nV_o <- function(cellchat, measure='count', LC="HSPC", RC="CRPC"){
    mat <- cellchat@net[[RC]][[measure]] - cellchat@net[[LC]][[measure]]
    mat
}

f_CC_nV_df <- function(sources=T, ...){
    mat <- f_CC_nV_o(...)
    if(sources){
        for(NM in rownames(mat)){
            f_CC_nV_split_s(mat, sources.use = NM)
        }
    }else{
        for(NM in colnames(mat)){
            f_CC_nV_split_t(mat, targets.use = NM)
        }
    }
}
```

## 第二步 绘图

```r
pdf(file = 'nV_df.pdf', height = 32, width = 16)
par(mfrow = c(3,2), xpd=TRUE)
f_CC_cmp_nV_df(SS)
f_CC_cmp_nV_df(ER)
f_CC_cmp_nV_df(CC)
dev.off()

pdf(file = 'nV_chord_gene_fF_1.pdf', height = 48, width = 24)
par(mfrow = c(3,2), xpd=TRUE)
f_CC_cmp_nV_chord_gene(SS_l, SS, sources.use = c('Fibroblasts'), targets.use = c(1:6),title.name = 'SS:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(ER_l, ER, sources.use = c('Fibroblasts'), targets.use = c(1:6),title.name = 'ER:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(CC_l, CC, sources.use = c('Fibroblasts'), targets.use = c(1:6),title.name = 'CC:Signaling from Fibroblasts - ')
dev.off()

pdf(file = 'nV_chord_gene_fF_2.pdf', height = 48, width = 24)
par(mfrow = c(3,2), xpd=TRUE)
f_CC_cmp_nV_chord_gene(SS_l, SS, sources.use = c('Fibroblasts'), targets.use = c(7:12),title.name = 'SS:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(ER_l, ER, sources.use = c('Fibroblasts'), targets.use = c(7:12),title.name = 'ER:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(CC_l, CC, sources.use = c('Fibroblasts'), targets.use = c(7:12),title.name = 'CC:Signaling from Fibroblasts - ')
dev.off()

pdf(file = 'nV_chord_gene_fF_3.pdf', height = 48, width = 24)
par(mfrow = c(3,2), xpd=TRUE)
f_CC_cmp_nV_chord_gene(SS_l, SS, sources.use = c('Fibroblasts'), targets.use = c(13:18),title.name = 'SS:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(ER_l, ER, sources.use = c('Fibroblasts'), targets.use = c(13:18),title.name = 'ER:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(CC_l, CC, sources.use = c('Fibroblasts'), targets.use = c(13:18),title.name = 'CC:Signaling from Fibroblasts - ')
dev.off()

pdf(file = 'nV_chord_gene_fF_4.pdf', height = 48, width = 24)
par(mfrow = c(3,2), xpd=TRUE)
f_CC_cmp_nV_chord_gene(SS_l, SS, sources.use = c('Fibroblasts'), targets.use = c(19:24),title.name = 'SS:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(ER_l, ER, sources.use = c('Fibroblasts'), targets.use = c(19:24),title.name = 'ER:Signaling from Fibroblasts - ')
f_CC_cmp_nV_chord_gene(CC_l, CC, sources.use = c('Fibroblasts'), targets.use = c(19:24),title.name = 'CC:Signaling from Fibroblasts - ')
dev.off()

pdf(file = 'nV_df_SS_s_c.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = T, cellchat = SS)
dev.off()

pdf(file = 'nV_df_SS_s_w.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = T, cellchat = SS, measure = "weight")
dev.off()

pdf(file = 'nV_df_SS_t_c.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = F, cellchat = SS)
dev.off()

pdf(file = 'nV_df_SS_t_w.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = F, cellchat = SS, measure = "weight")
dev.off()

pdf(file = 'nV_df_ER_s_c.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = T, cellchat = ER)
dev.off()

pdf(file = 'nV_df_ER_s_w.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = T, cellchat = ER, measure = "weight")
dev.off()

pdf(file = 'nV_df_ER_t_c.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = F, cellchat = ER)
dev.off()

pdf(file = 'nV_df_ER_t_w.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = F, cellchat = ER, measure = "weight")
dev.off()

pdf(file = 'nV_df_CC_s_c.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = T, cellchat = CC)
dev.off()

pdf(file = 'nV_df_CC_s_w.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = T, cellchat = CC, measure = "weight")
dev.off()

pdf(file = 'nV_df_CC_t_c.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = F, cellchat = CC)
dev.off()

pdf(file = 'nV_df_CC_t_w.pdf', height = 36, width = 24)
par(mfrow = c(6,4), xpd=TRUE)
f_CC_nV_df(sources = F, cellchat = CC, measure = "weight")
dev.off()
```