---
title: scMetabolism分析细胞代谢
tags: []
id: '1539'
categories:
  - - 单细胞下游分析
date: 2022-02-09 03:53:56
---

## 包安装

[https://github.com/wu-yc/scMetabolism](https://github.com/wu-yc/scMetabolism)

## 第一步 分离对比对象

```r
HSPC <- subset(immune, group == 'HSPC')
CPRC <- subset(immune, group == 'CRPC')
countexp_HSPC <-sc.metabolism.Seurat(obj = HSPC, method = "VISION", imputation = F, ncores = 12, metabolism.type = "KEGG")
countexp_CPRC <-sc.metabolism.Seurat(obj = CPRC, method = "VISION", imputation = F, ncores = 12, metabolism.type = "KEGG")
```

## 第二步 获得代谢矩阵

```r
etabolism.matrix_HSPC <- countexp_HSPC@assays$METABOLISM$score
etabolism.matrix_CPRC <- countexp_CPRC@assays$METABOLISM$score
```

## 第三步 代谢差异分析

```r
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
    res[['def']] <- with(res, meanE - meanN)
    res
}
f_metaN2cellN <- function(scRNA, metaN, typeN, matrixN=F){
    res <- scRNA[[metaN]]
    idx <- which(res[[1]] == typeN)
    if(matrixN){
        return(gsub('-','.',rownames(res)[idx]))
    }else{
        return(rownames(res)[idx])
    }
}
f_DEmetabolism <- function(countexpN, countexpE, metaN, typeN){
    matrixN <- countexpN@assays$METABOLISM$score
    matrixE <- countexpE@assays$METABOLISM$score
    matrixN <- matrixN[,f_metaN2cellN(countexpN, metaN, typeN, matrixN = T)]
    matrixE <- matrixE[,f_metaN2cellN(countexpE, metaN, typeN, matrixN = T)]
    f_DEmetabolism_01(matrixN, matrixE)
}
```

## 第四步 代谢差异展示

```r
require(ggrepel)
f_ggplot2_ti <- function(p, title, subtitle=''){
    if (subtitle == ''){
        (p + ggtitle(title) + theme(plot.title = element_text(hjust = 0.5)))
    }else{
        (p + ggtitle(title, subtitle=subtitle) + 
         theme(plot.title = element_text(hjust = 0.5), plot.subtitle = element_text(hjust = 0.5)))
    }
}
f_DEmetabolism_Volcano <- function(countexpN, countexpE, metaN, typeN, delta = 1, Threshold_p = 0.05, lc_rep=5){
    DEmm <- f_DEmetabolism(countexpN, countexpE, metaN, typeN)
    col_vector = rep(rgb(108, 200, 228, maxColorValue = 255), nrow(DEmm))
    Threshold = with(DEmm, delta*sd(abs(def)))
    
    col_vector[with(DEmm, `p-value` < Threshold_p & def > Threshold)] = rgb(226, 61, 75, maxColorValue = 255)
    col_vector[with(DEmm, `p-value` < Threshold_p & def < -Threshold)] = rgb(232, 168, 71, maxColorValue = 255)
    
    DEmm[['Name']] <- rownames(DEmm)
    DEmm[['col']] <- col_vector
    DEmm[['-log10(P)']] <- with(DEmm, -log10(`p-value`))
    
    lc_tp_logFC <- DEmm$def
    lc_tp_logFC[with(DEmm, `p-value` >= Threshold_p)] = 0
    lc_idx <- order(lc_tp_logFC)[c(1:lc_rep, (nrow(DEmm)+1-lc_rep):nrow(DEmm))]
    
    res <- ggplot() + geom_point(aes(def, `-log10(P)`, col=I(col)), data = DEmm) 
    res <- res + theme_bw() + theme(panel.grid=element_line(colour=NA))
    res <- res + geom_hline(yintercept=-log10(Threshold_p), linetype="longdash")
    res <- res + geom_vline(xintercept=c(Threshold, -Threshold), linetype="longdash")
    res <- res + geom_text_repel(data=DEmm[lc_idx,],aes(def,`-log10(P)`,label=Name), force=T, max.overlaps=Inf)
    f_ggplot2_ti(res, typeN)
}
```

## 第五步 全部展示

```r
require(export)
f_ppt_output <- function(fileName, image){
    for (p in image){
        graph2ppt(x=p, file=fileName, width=9, aspectr=1.5, append = TRUE)
    }
}
pl <- list()
for (tN in unique(unlist(countexp_HSPC[['immune_type_2']]))){
    pl[[tN]] <- f_DEmetabolism_Volcano(countexp_HSPC, countexp_CPRC, 'immune_type_2', tN)
}
f_ppt_output('HSPC vs CRPC.pptx', pl)

input.pathway<- rownames(countexp_HSPC@assays$METABOLISM$score)
p1 <- DotPlot.metabolism(obj = countexp_HSPC, pathway = input.pathway, phenotype = "immune_type_2", norm = "y") + DotPlot.metabolism(obj = countexp_CPRC, pathway = input.pathway, phenotype = "immune_type_2", norm = "y")
ggsave(p1, filename = 'HSPC vs CRPC.pdf', dpi = 1200, width = 24, height = 36, device = 'pdf', limitsize = FALSE)
```

## 第六步 基因热图-获取代谢相关基因

```r
require(KEGGREST)
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
MMPath <- keggGet('hsa01100')
MMPath <- names(MMPath[[1]]$REL_PATHWAY)
MMgene <- f_KEGG_id2symbol(MMPath)
```

## 第六步 绘制基因热图

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
        res[[name]] <- rowMeans(matrixA[,group[[name]]])
    }
    res
}
require(reshape2)
f_matrix_heatmap <- function(dfA){
    # 标准化一下
#     dfA <- as.data.frame(scale(dfA))
    # 转换前，先增加一列ID列，保存行名字
    dfA$df_ID <- rownames(dfA)
    dfm <- melt(dfA, na.rm = T, id.vars = c('df_ID'))
    p <- ggplot(dfm, aes(x=variable, y=df_ID)) 
    p <- p + geom_tile(aes(fill=value))
    p <- p + theme(axis.text.x=element_text(angle=45, hjust=1, vjust=1))
    p <- p + scale_fill_gradient(low = "blue", high = "pink")
    p <- p + xlab("samples") + theme_bw() + theme(panel.grid.major = element_blank()) + theme(legend.key=element_blank())
    p
}

mono <- subset(immune, immune_type_2 %in% c('TAM','monocytes', 'S100A8+ monocytes'))
mono <- sc.metabolism.Seurat(obj = mono, method = "VISION", imputation = F, ncores = 12, metabolism.type = "KEGG")
a <- f_matrix_groupMean(mono@assays$METABOLISM$score, mono[['patient_id']])
p <- f_matrix_heatmap(a)
ggsave(p, filename = 'mono_HSPC vs CRPC_heatmap.pdf', dpi = 1200, width = 8, height = 24, device = 'pdf', limitsize = FALSE)
```

## 第七步 绘制交叉热图

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
f_matrix_heatmap <- function(dfA){
    # 标准化一下
#     dfA <- as.data.frame(scale(dfA))
    # 转换前，先增加一列ID列，保存行名字
    dfA$df_ID <- rownames(dfA)
#     dfA$df_ID <- factor(x = dfA$df_ID, ordered = T)
    dfm <- melt(dfA, na.rm = T, id.vars = c('df_ID'))
    dfm$variable <- factor(x = as.character(dfm$variable), ordered = T)
    p <- ggplot(dfm, aes(x=variable, y=df_ID)) 
    p <- p + geom_tile(aes(fill=value))
    p <- p + scale_fill_gradient(low = "white", high = "red")
    p <- p + xlab("samples") + theme_bw() + theme(panel.grid.major = element_blank()) + theme(legend.key=element_blank())
    p <- p + theme(axis.text.x=element_text(angle=45, hjust=1, vjust=1))
    p
}
```

```r
immune[['tmp2']] <- paste(as.character(immune[['immune_type_2']][[1]]), as.character(immune[['patient_id']][[1]]))

tmp2 <- f_matrix_groupMean(immune@assays$METABOLISM$score[unlist(sR),], immune[['tmp2']])
p2 <- f_matrix_heatmap(tmp2)
ggsave(p2, filename = 'HSPC vs CRPC_heatmap_2.pdf', dpi = 1200, width = 24, height = 24, device = 'pdf', limitsize = FALSE)
```