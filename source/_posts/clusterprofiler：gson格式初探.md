---
title: clusterProfiler：gson格式初探
tags: []
id: '1922'
categories:
  - - uncategorized
date: 2022-07-20 22:43:59
---

## 更新包

*   [conda activate clusterprofile](https://occdn.limour.top/1617.html)
*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://github.com/YuLab-SMU/clusterProfiler/archive/refs/heads/master.zip -O clusterProfiler-master.zip
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://github.com/YuLab-SMU/DOSE/archive/refs/heads/master.zip -O DOSE-master.zip
*   devtools::install\_local('DOSE-master.zip')
*   devtools::install\_local('clusterProfiler-master.zip')

## 保存gson到本地

*   library(clusterProfiler)
*   kk <- gson\_KEGG('hsa')
*   gson::write.gson(kk, file = './KEGG.hsa.gson')

## 使用本地gson进行KEGG分析

*   library(clusterProfiler)
*   kk <- gson::read.gson('KEGG.hsa.gson')
*   rres <- readRDS('deg.rds')
*   rres <- rres\[order(rres$log2FoldChange),\]

```R
require(org.Hs.eg.db)
f_id2name_kk <- function(lc_cgene, keytype="SYMBOL", columns="ENTREZID"){
    res=AnnotationDbi::select(org.Hs.eg.db::org.Hs.eg.db,keys=lc_cgene,columns=columns, keytype=keytype)
    res <- subset(res, !is.na(ENTREZID))
    res$ENTREZID
}
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
kegg.d <- enrichKEGG(gene = f_id2name_kk(subset(rres, log2FoldChange < -0.1 & padj < 0.05)$symbol),
      organism = kk,
      universe = f_id2name_kk(rres$symbol),
      pAdjustMethod = "fdr",
      qvalueCutoff =0.05)
f_kegg_p(kegg.d, n =15) %>% f_title("kegg.d")
```