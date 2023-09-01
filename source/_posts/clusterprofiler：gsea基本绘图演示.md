---
title: clusterProfiler：GSEA基本绘图演示
tags: []
id: '1927'
categories:
  - - uncategorized
date: 2022-07-22 15:08:54
---

## GSEA分析

```R
require(clusterProfiler)
require(enrichplot)
rres <- readRDS('DEGs_X1.OE.DMSO_X2.OE.DMSO_vs._X1.control.DMSO_X2.control.DMSO_DESeq2.rds')
lgene <- rres$log2FoldChange
names(lgene) <- rres$symbol
WP <- readRDS('WP.hsa.rds')
WP$TERM2NAME$name <- gsub('%.*%Homo sapiens', '', WP$TERM2NAME$name)
set.seed(0)
gse.WP <- GSEA(gene = lgene,
             pAdjustMethod = "fdr",
             eps = 0,
             pvalueCutoff = 0.1,
             TERM2GENE = WP$TERM2GENE,
             TERM2NAME = WP$TERM2NAME)
require(tidyverse)
require(enrichplot)
require(DOSE)
f_kegg_p <- function(keggr2, n = 15){
    keggr <- subset(keggr2@result, p.adjust < 0.05)
    keggr[['-log(Padj)']] <- -log10(keggr[['p.adjust']])
    keggr[['geneRatio']] <- parse_ratio(keggr[['GeneRatio']])
    keggr$Description <- factor(keggr$Description, 
                                       levels=keggr[order(keggr$geneRatio),]$Description)
    ggplot(head(keggr,n),aes(x=geneRatio,y=Description))+
        geom_point(aes(color=`-log(Padj)`,
                 size=`Count`))+
        theme_bw()+
        scale_color_gradient(low="blue1",high="brown1")+
        labs(y=NULL) + 
        theme(axis.text.x=element_text(angle=90,hjust = 1,vjust=0.5, size = 12),
             axis.text.y=element_text(size = 15))
}
f_title <- function(gp,title){
    gp + labs(title = title) + theme(plot.title = element_text(hjust = 0.5))
}
gse.WP@result
```

## 通路总图

```R
require(ggplot2)
require(stringr)
options(repr.plot.width=8, repr.plot.height=8)
gse.WP@result['Ratio'] <- with(gse.WP@result, ((str_count(core_enrichment, '/') + 1) / setSize))
gse.WP@result$Description <- with(gse.WP@result,factor(Description, levels = Description[order(NES)]))
p <- ggplot(subset(gse.WP@result, NES>0),aes(NES,Description)) 
p <- p + geom_point(aes(size=Ratio,color=-log10(qvalues))) + scale_size(range = c(5,10))
p <- p + scale_color_continuous(low=rgb(1,0,0,0.5) ,high=rgb(0,0,1,0.5))
p <- p + theme_bw() + theme(axis.text=element_text(size=20),axis.title=element_text(size=20))
p <- p + scale_y_discrete(labels=function(y){str_wrap(y,width = 20)})
p
```

## GSEA图

```R
options(repr.plot.width=8, repr.plot.height=8)
p <- gseaplot2(gse.WP, geneSetID=27, color = 'gray', ES_geom = 'dot', base_size = 20)
p
```

## 通路网络图

```R
gse.WP2 <- pairwise_termsim(gse.WP)
p <- emapplot(gse.WP2, showCategory=subset(gse.WP@result, NES>0)$Description)
p
```

## 通路基因图

```R
#  Layout of the map, e.g. 'star', 'circle', 'gem', 'dh', 'graphopt', 'grid', 'mds', 'randomly', 'fr', 'kk', 'drl' or 'lgl'.
options(repr.plot.width=8, repr.plot.height=8, ggrepel.max.overlaps = Inf)
p <- cnetplot(gse.WP2, foldChange=lgene, showCategory=subset(gse.WP@result, NES>0)$Description, layout = "dh")
p
```