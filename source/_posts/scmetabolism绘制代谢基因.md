---
title: scMetabolism绘制代谢基因
tags: []
id: '1566'
categories:
  - - R绘图奇技淫巧
date: 2022-02-10 16:23:32
---

## 定义绘图函数

```r
# 绘制热图
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

## 获取绘图数据

```r
immune <- readRDS('../figure1.cellTypeist/immune.rds')
immune[['tmp']] <- paste(as.character(immune[['immune_type_2']][[1]]), as.character(immune[['group']][[1]]))

M_c <- subset(immune, cell_type == 'Myeloid')
L_c <- subset(immune, cell_type != 'Myeloid')

M_m <- read.csv('selected_for_gene_M.CSV', header = F)
L_m <- read.csv('selected_for_gene_L.CSV', header = F)

keggL <- keggList("pathway","hsa")

M_g <- f_KEGG_name2id(keggL, unlist(M_m))
M_g <- f_KEGG_id2symbol(M_g)
M_g <- M_g[M_g %in% rownames(M_c)]

L_g <- f_KEGG_name2id(keggL, unlist(L_m))
L_g <- f_KEGG_id2symbol(L_g)
L_g <- L_g[L_g %in% rownames(L_c)]
```

## 绘图并保存

```r
M_p <- f_matrix_heatmap(f_matrix_groupMean(M_c@assays$RNA@data[M_g,], M_c[['tmp']], matrixN = F), levels = M_g)

ggsave(M_p, filename = 'scMetabolism_gene_M.pdf', dpi = 1200, width = 4, height = 32, device = 'pdf', limitsize = FALSE)

L_p <- f_matrix_heatmap(f_matrix_groupMean(L_c@assays$RNA@data[L_g,], L_c[['tmp']], matrixN = F), levels = L_g)

ggsave(L_p, filename = 'scMetabolism_gene_L.pdf', dpi = 1200, width = 6, height = 48, device = 'pdf', limitsize = FALSE)
```