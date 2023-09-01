---
title: clusterProfiler：构建MsigDB的数据库
tags: []
id: '1949'
categories:
  - - uncategorized
date: 2022-08-07 09:44:47
---

[clusterProfiler：自定义数据库](https://occdn.limour.top/2128.html)中展示过构建[HALLMARKS](http://www.gsea-msigdb.org/gsea/downloads.jsp)的方式，但是速度有点慢，这里简单改动一下，对[C5\_HPO](https://www.gsea-msigdb.org/gsea/msigdb/collections.jsp#C5)进行构建。

## 构建

```R
text <- readLines("c5.hpo.v7.5.1.symbols.gmt", encoding = "UTF-8")
text <- strsplit(text, "\t")
H.ALL <- list(TERM2GENE=data.frame(), TERM2NAME=data.frame())
for (i in 1:length(text)){
    line <- text[[i]]
    gsid <- line[1]
    H.ALL$TERM2NAME <- rbind(H.ALL$TERM2NAME, c(i,gsid))
    lc_gsid <- rep(x = i, times = (length(line)-2))
    H.ALL$TERM2GENE <- rbind(H.ALL$TERM2GENE, cbind(lc_gsid,line[3:length(line)]))
}
colnames(H.ALL$TERM2NAME) <- c('gsid', 'name')
colnames(H.ALL$TERM2GENE) <- c('gsid', 'gene')
saveRDS(H.ALL, 'C5.HPO.rds')
```

## 富集

```R
require(tidyverse)
require(enrichplot)
require(DOSE)
require(stringr)
require(clusterProfiler)
f_kegg_p <- function(keggr2, n = 15){
    keggr <- subset(keggr2@result, p.adjust < 0.1)
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
             axis.text.y=element_text(size = 15))+
     scale_size(range = c(5,10)) + 
     theme(axis.text=element_text(size=20),axis.title=element_text(size=20)) + 
     scale_y_discrete(labels=function(y){str_wrap(gsub('_', ' ', y),width = 20)})
}
f_title <- function(gp,title){
    gp + labs(title = title) + theme(plot.title = element_text(hjust = 0.5))
}
```

```R
KEGG <- readRDS('../MsigDB/C5.HPO.rds')
set.seed(0)
res <- enricher(gene = DEGPH$Name, 
             pAdjustMethod = "fdr",
             pvalueCutoff = 0.1,
             qvalueCutoff = 0.1,
             TERM2GENE = KEGG$TERM2GENE,
             TERM2NAME = KEGG$TERM2NAME)
options(repr.plot.width=8, repr.plot.height=12)
f_kegg_p(res, n =15) %>% f_title("C42vsLNCaP_EtOH_C5_HPO")
```

[GSEA以及其他高级绘图](https://occdn.limour.top/2142.html)，注：GESA添加`nPermSimple = 100000`可以提高P值计算准确性

## 通过预分配的方式进行构建

```R
text <- readLines("c5.all.v7.5.1.symbols.gmt", encoding = "UTF-8")
text <- strsplit(text, "\t")
pre_len <- 0
for (i in 1:length(text)){
line <- text[[i]]
    pre_len <- pre_len + (length(line)-2)
}
H.ALL <- list(TERM2NAME=data.frame(), TERM2GENE=data.frame(gsid=rep(NA,pre_len), gene=rep(NA,pre_len)))
pre_tmp <- 0
for (i in 1:length(text)){
    line <- text[[i]]
    gsid <- line[1]
    H.ALL$TERM2NAME <- rbind(H.ALL$TERM2NAME, c(i,gsid))
    pre_genelen <- (length(line)-2)
    lc_gsid <- rep(x = i, times = pre_genelen)
    H.ALL$TERM2GENE[(pre_tmp+1):(pre_tmp + pre_genelen),] <- cbind(lc_gsid,line[3:length(line)])
    pre_tmp <- pre_tmp + pre_genelen
}
colnames(H.ALL$TERM2NAME) <- c('gsid', 'name')
colnames(H.ALL$TERM2GENE) <- c('gsid', 'gene')
saveRDS(H.ALL, 'C5.all.rds')
```