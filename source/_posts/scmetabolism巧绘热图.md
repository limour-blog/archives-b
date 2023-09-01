---
title: scMetabolism巧绘热图
tags: []
id: '1602'
categories:
  - - R绘图奇技淫巧
date: 2022-02-12 13:03:38
---

## 定义汇总函数

```r
f_metaG2G <- function(metaG, matrixN=F, downSample=NULL){
    res  <- list()
    alltype <- unique(metaG[[1]])
    for(type in alltype){
        res[[type]] <- rownames(metaG)[metaG[[1]] == type]
        if (matrixN){
            res[[type]] <- gsub('-','.',res[[type]])
        }
    }
    if(!is.null(downSample)){
        pTb <- f_br_cluster(NULL, downSample, metaG)
        for (nM in colnames(pTb)){
            tmp <- pTb[nM]
            pTb[nM][tmp>0] <- min(tmp[tmp>0])
        }
        pTb <- apply(X = pTb, FUN = max, MARGIN=1)
        for(nM in names(pTb)){
            res[[nM]] <- sample(x = res[[nM]], size = pTb[nM])
        }
    }
    res
}
f_br_cluster_f <- function(sObject, lc_groupN){
    if(is.null(sObject)){
        lc_filter <- unlist(unique(lc_groupN))
    }else{
        lc_filter <- unlist(unique(sObject[[lc_groupN]]))
    }
    lc_filter <- lc_filter[!is.na(lc_filter)]
    lc_filter
}
 
f_br_cluster <- function(sObject, lc_groupN, lc_labelN, lc_prop = F){
    if(!is.null(sObject)){
        lc_g <- f_metaG2G(sObject[[lc_groupN]])
        lc_l <- sObject[[lc_labelN]]
    }else{
        lc_g <- f_metaG2G(lc_groupN)
        lc_l <- lc_labelN
    }
    lc_l[[1]] <- as.character(lc_l[[1]])
    res <- data.frame(row.names = f_br_cluster_f(sObject, lc_labelN))
    if(lc_prop){
        for(Nm in names(lc_g)){
            tmp <- prop.table(table(lc_l[lc_g[[Nm]],]))
            res[[Nm]] <- tmp[rownames(res)]
        }
    }else{
        for(Nm in names(lc_g)){
            tmp <- table(lc_l[lc_g[[Nm]],])
            res[[Nm]] <- tmp[rownames(res)]
        }
    }
    res[is.na(res)] = 0
    res
}
f_rank_transformation_o <- function(lo){
    lo <- unlist(lo)
    idx <- order(lo, decreasing = F)
    idm <- as.data.frame(table(lo))
    idm <- idm[idm$Freq>1, ]
    idm <- idm[[1]]
    for(i in idm){
        ii <- (lo == i)
        idx[ii] <- mean(idx[ii])
    }
    names(idx) <- names(lo)
    idx
}
f_rank_transformation <- function(df){
    as.data.frame(t(apply(X = df, FUN = f_rank_transformation_o, MARGIN = 1)))
}
```

```r
f_scale_t <- function(matrixA){
    t(scale(t(as.matrix(matrixA))))
}
f_matrix_groupMean <- function(matrixA, group, matrixN = T, normal_distribution=F, downSample=NULL, autoG2G=T, scale=F){
    if(!matrixN){
        matrixA <- as.data.frame(as.matrix(matrixA))
    }
    res <- data.frame(row.names = rownames(matrixA))
    if(autoG2G){
        group <- f_metaG2G(group, matrixN = matrixN, downSample = downSample)
    }
    matrixA <- matrixA[, Reduce(x = group, f = c)]
    if(!normal_distribution){
        matrixA <- f_rank_transformation(matrixA)
    }else{
        if(scale){
            matrixA <- f_scale_t(matrixA)
        }
    }
    rowF <- rowMeans
    for(name in names(group)){
        if (length(group[[name]]) == 1){
            res[[name]] <- matrixA[,group[[name]]]
        }else{
            res[[name]] <- rowF(matrixA[,group[[name]]])
        }
    }
    res
}
f_matrix_groupMean_g <- function(groupN, matrixA, group, matrixN = T, normal_distribution=F, autoG2G=T, scale=F){
    if(!matrixN){
        matrixA <- as.data.frame(as.matrix(matrixA))
    }
    res <- data.frame(row.names = rownames(matrixA))
    if(autoG2G){
        group <- f_metaG2G(group, matrixN = matrixN, downSample = downSample)
    }
    group <- Reduce(x = group, f = c)
    matrixA <- matrixA[, group]
    if(!normal_distribution){
        matrixA <- f_rank_transformation(matrixA)
    }else{
        if(scale){
            matrixA <- f_scale_t(matrixA)
        }
    }
    groupN <- f_metaG2G(groupN, matrixN = matrixN)
    rowF <- rowMeans
    for(name in names(groupN)){
        tmp <- (group %in% groupN[[name]])
        if (sum(tmp) == 1){
            res[[name]] <- matrixA[,tmp]
        }else{
            res[[name]] <- rowF(matrixA[,tmp])
        }
    }
    res
}
```

```r
# 提取代谢基因
require(stringr)
f_KEGG_name2id <- function(keggL, nameL){
    res <- NULL
    for (i in 1:length(nameL)){
        fuck <- gsub(pattern = '\\(', replacement = '\\\\(', x = nameL[[i]])
        fuck <- gsub(pattern = '\\)', replacement = '\\\\)', x = fuck)
        idx <- str_detect(keggL, fuck)
        tmp <- keggL[idx]
        if (length(tmp) == 1){
            tmp <- names(tmp)
            tmp <- substr(x =tmp, start = 6, stop=15)
            res <- c(res, tmp)
        }
        if (length(tmp) > 1){
            print(tmp)
        }
    }
    res
}
require(KEGGREST)
f_KEGG_id2symbol <- function(KEGGid){
    res <- NULL
    for (hsa_id in KEGGid){
        hsa_info <- keggGet(hsa_id)
        hsa_info <- hsa_info[[1]]$GENE
        hsa_info <- hsa_info[seq(from = 2, to = length(hsa_info), by = 2)]
        hsa_info <- strsplit(hsa_info,split = ';')
        hsa_info <- unlist(lapply(hsa_info, function(X){X[1]}))
        res <- c(res, hsa_info)
    }
    res <- unique(res)
    res <- res[str_detect(res, pattern = '\\] \\[', negate = T)]
    res
}
```

## 定义统计函数

```r
require(fdrtool)
f_DEmetabolism_01 <- function(matrixN, matrixE){
    res <- data.frame(row.names = rownames(matrixN))
    res[['meanN']] = 0
    res[['meanE']] = 0
    res[['p-value']] = 1
    for(i in 1:nrow(matrixN)){
        test <- t.test(unlist(matrixN[i,]),unlist(matrixE[i,]))
        res[i, 'p-value'] = test$p.value
        res[i, 'meanN'] = test$estimate[1]
        res[i, 'meanE'] = test$estimate[2]
    }
    res <- res[!is.na(res$`p-value`), ]
    res[['dif']] <- with(res, meanE - meanN)
    tmp <- fdrtool(res$`p-value`,statistic="pvalue")
    res[['qval']] <- tmp$qval
    res[['lfdr']] <- tmp$lfdr
    res
}
f_DEmetabolism_02 <- function(matrixN, matrixE){
    res <- data.frame(row.names = rownames(matrixN))
    for(i in 1:nrow(matrixN)){
        test <- wilcox.test(unlist(matrixN[i,]),unlist(matrixE[i,]))
        res[i, 'p-value'] = test$p.value
        res[i, 'W'] = test$statistic[[1]]
    }
    res <- res[!is.na(res$`p-value`), ]
    tmp <- f_rank_transformation(cbind(matrixN, matrixE))
    res[['MRN']] <- rowMeans(tmp[colnames(matrixN)])
    res[['MRE']] <- rowMeans(tmp[colnames(matrixE)])
    res[['dif']] <- with(res, MRE - MRN)
    tmp <- fdrtool(res$`p-value`,statistic="pvalue")
    res[['qval']] <- tmp$qval
    res[['lfdr']] <- tmp$lfdr
    
    res
}

f_DEmetabolism <- function(countexp, group, groupN, normal_distribution=F, matrixN=T){
    if(matrixN){
        matrixA <- countexp@assays$METABOLISM$score
    }else{
        matrixA <- as.data.frame(as.matrix(countexp@assays$RNA@data))
    }
    group <- Reduce(x = group, f = c)
    groupN <- f_metaG2G(countexp[[groupN]], matrixN = matrixN)
    matrixN <- group[group %in% groupN[[1]]]
    matrixE <- group[group %in% groupN[[2]]]
    matrixN <- matrixA[,matrixN]
    matrixE <- matrixA[,matrixE]
    if(normal_distribution){
        f_DEmetabolism_01(matrixN, matrixE)
    }else{
        f_DEmetabolism_02(matrixN, matrixE)
    }
}
```

## 定义绘图函数

```r
require(reshape2)
require(ggplot2)
f_matrix_heatmap <- function(dfA, levels = NULL, xlevels=NULL){
    # 转换前，先增加一列ID列，保存行名字
    dfA <- as.data.frame(dfA)
    dfA$df_ID <- rownames(dfA)
    dfm <- melt(dfA, na.rm = T, id.vars = c('df_ID'))
    if(is.null(xlevels)){
        dfm$variable <- factor(x = as.character(dfm$variable), ordered = T)
    }else{
        dfm$variable <- factor(x = as.character(dfm$variable), levels = xlevels)
    }
    if (length(levels) > 0){
        dfm$df_ID  <- factor(x = as.character(dfm$df_ID), levels = rev(levels))
    }
    p <- ggplot(dfm, aes(x=variable, y=df_ID)) 
    p <- p + geom_tile(aes(fill=value))
    p <- p + scale_fill_gradient(low = 'white', high = 'red')
#     p <- p + scale_fill_gradient(low = 'steel blue', high = 'pink')
#     p <- p + scale_fill_gradientn(colours = c('#3E5CC5','#65B48E','#E6EB00','#E64E00'))
    p <- p + xlab("samples") + theme_bw() + theme(panel.grid.major = element_blank()) + theme(legend.key=element_blank())
    p <- p + theme(axis.text.x=element_text(angle=45, hjust=1, vjust=1))
    p <- p + labs(x=NULL, y=NULL) # 删除xy轴标题 
    p
}
```

## 热图绘制

```r
grp <- f_metaG2G(metaG = F_c[['patient_id']], downSample = F_c[['group']], matrixN = T)
sR_d <- f_DEmetabolism(F_c, grp, 'group')
sR_d <- sR_d[order(sR_d$dif, decreasing = T),]
sR_d <- sR_d[sR_d$lfdr<0.00001,]
write.csv(sR_d, file='05.HSPC vs CRPC_F.csv')
sR <- rownames(sR_d)

xlev = c('patient1','patient5','patient3', 'patient4')
p <- f_matrix_heatmap(scale(f_matrix_groupMean(F_c@assays$METABOLISM$score[unlist(sR),], group = grp, autoG2G = F)), levels = unlist(sR), xlevels = xlev)
ggsave(p, filename = '05.HSPC vs CRPC_F.pdf', dpi = 1200, width = 12, height = 10, device = 'pdf', limitsize = FALSE)

p <- f_matrix_heatmap(scale(f_matrix_groupMean_g(F_c[['group']],F_c@assays$METABOLISM$score[unlist(sR),], group = grp, autoG2G = F)), levels = unlist(sR))
ggsave(p, filename = '05.HSPC vs CRPC_F_g.pdf', dpi = 1200, width = 6, height = 6, device = 'pdf', limitsize = FALSE)
```

```r
keggL <- keggList("pathway","hsa")
F_m <- openxlsx::read.xlsx(xlsxFile = "../01/HSD17B2_selected_for_gene_heatmap.xlsx", sheet = 1, colNames = F)
M_g <- f_KEGG_name2id(keggL, unlist(F_m))
M_g <- f_KEGG_id2symbol(M_g)
M_g <- M_g[M_g %in% rownames(F_c)]
grp_ <- f_metaG2G(metaG = F_c[['patient_id']], downSample = F_c[['group']], matrixN = F)
sR_d <- f_DEmetabolism(F_c[M_g,], grp_, 'group', normal_distribution = T, matrixN = F)
sR_d <- sR_d[order(sR_d$dif, decreasing = T),]
sR_d <- sR_d[sR_d$lfdr<0.001,]
write.csv(sR_d, file='05.HSPC vs CRPC_F_gene_H.csv')
sR <- rownames(sR_d)

xlev = c('patient1','patient5','patient3', 'patient4')
p <- f_matrix_heatmap(scale(f_matrix_groupMean(F_c@assays$RNA@data[unlist(sR),], group = grp_, autoG2G = F, normal_distribution = T, matrixN = F)), levels = unlist(sR), xlevels = xlev)
ggsave(p, filename = '05.HSPC vs CRPC_F_gene_H.pdf', dpi = 1200, width = 4, height = 10, device = 'pdf', limitsize = FALSE)

p <- f_matrix_heatmap(scale(f_matrix_groupMean_g(F_c[['group']],F_c@assays$RNA@data[unlist(sR),], group = grp_, autoG2G = F, normal_distribution = T, matrixN = F)), levels = unlist(sR))
ggsave(p, filename = '05.HSPC vs CRPC_F_g_gene_H.pdf', dpi = 1200, width = 3, height = 6, device = 'pdf', limitsize = FALSE)
```