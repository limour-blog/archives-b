---
title: 蛋白质组RPPA抗体与基因信息映射
tags: []
id: '2021'
categories:
  - - uncategorized
date: 2022-09-11 09:13:59
---

之前通过[TCGAbiolinks下载了蛋白质组数据](https://occdn.limour.top/2325.html)，而其中**peptide\_target**给的是抗体的名字，进行富集时需要进行ID转换，我们先来构造一个映射表。RPPA的抗体与基因映射关系在[这里](https://www.mdanderson.org/research/research-resources/core-facilities/functional-proteomics-rppa-core/antibody-information-and-protocols.html)。

## 准备映射表

从[官网](https://www.mdanderson.org/research/research-resources/core-facilities/functional-proteomics-rppa-core/antibody-information-and-protocols.html)下载手工版的映射表，整理成下面的格式，RPPA\_Expanded\_Ab\_List\_Updated、RPPA\_Standard\_Ab\_List\_Updated和the list of [Updated Gene Names](https://www.mdanderson.org/content/dam/mdanderson/documents/core-facilities/Functional%20Proteomics%20RPPA%20Core%20Facility/Corrected_Gene_Names.pdf)都需要。

![](https://img-cdn.limour.top/2022/09/11/631d8cff8b914.png)

## 清洗合并映射表

```R
ad2g1 <- read.csv('sp_Ab2Gene.CSV')
ad2g2 <- read.csv('sp_Ab2Gene_old.CSV')
ad2g3 <- read.csv('sp_Ab2Gene_new.CSV')
ad2g <- ad2g2[(ad2g2$Ab %in% ad2g1$Ab) == F,]
ad2g <- rbind(ad2g, ad2g1, ad2g3)
ad2g <- unique(ad2g)
ad2g[order(ad2g$Gene),]
 
ad2g_d <- list()
ad2g_d$gene <- unique(ad2g$Gene)
ad2g_d$raw <- ad2g
ad2g_d$g2ad <- list()
for (gene in ad2g_d$gene){
    ad2g_d$g2ad[[gene]] <- with(ad2g, Ab[Gene == gene])
}
ad2g_d$g2ad_c <- list()
ad2g$Ab <- stringr::str_replace_all(string = tolower(ad2g$Ab), pattern = '(_-\\. /)', replacement = '')
for (gene in ad2g_d$gene){
    ad2g_d$g2ad_c[[gene]] <- with(ad2g, Ab[Gene == gene])
}
```

## 查看遗漏数据

```R
load('PRAD_TCPA_DE.rdata')
tmp <- stringr::str_replace_all(string = tolower(r1$peptide_target), pattern = '(_-\\. /)', replacement = '')
r1[(tmp %in% ad2g$Ab) == F,]
```

通过[搜索](https://bioinformatics.mdanderson.org/NGCHM/compendia/TCGA/html/tcga_rppa_acc_v2.0_protein_protein.html)补全遗漏的映射表，添加到映射表中，重新清洗合并映射表

## 更改富集数据库的SYMBOL为Antibody

```R
KEGG <- readRDS('../../../GSEA/kk_SYMBOL.rds')
tmp_1 <- KEGG$TERM2GENE[ KEGG$TERM2GENE$gene %in% ad2g_d$gene, ]
tmp_2 <- KEGG$TERM2GENE[ (KEGG$TERM2GENE$gene %in% ad2g_d$gene)==F, ]
tmp_3 <- data.frame()
for(i in 1:nrow(tmp_1)){
    tmp_4 <- ad2g_d$g2ad_c[[tmp_1[i, 'gene']]]
    tmp_5 <- rep(tmp_1[i, 'gsid'], length(tmp_4))
    tmp_6 <- data.frame(gsid=tmp_5, gene=tmp_4)
    tmp_3 <- rbind(tmp_3, tmp_6)
}
tmp_3 <- unique(tmp_3)
KEGG_ab <- list()
KEGG_ab$TERM2NAME <- KEGG$TERM2NAME[KEGG$TERM2NAME$gsid %in% unique(tmp_3$gsid),]
KEGG_ab$TERM2GENE <- tmp_3
saveRDS(KEGG_ab, 'KEGG_Ab.rds')
```