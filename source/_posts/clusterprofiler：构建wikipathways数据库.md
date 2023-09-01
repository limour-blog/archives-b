---
title: clusterProfiler：构建WikiPathways数据库
tags: []
id: '1924'
categories:
  - - uncategorized
date: 2022-07-21 01:37:22
---

[WikiPathways](https://www.wikipathways.org/)是一个开放协作平台，旨在促进生物学界对通路信息的贡献和维护。它提供了一种新的模型，可以增强和补充[KEGG](http://www.genome.jp/kegg/)、[Reactome](http://www.reactome.org/)和[Pathway Commons](http://www.pathwaycommons.org/pc/)等正在进行的工作。

## 安装相应的R包

*   [conda activate clusterprofiler](https://occdn.limour.top/2126.htm)
*   conda install -c bioconda bioconductor-rwikipathways -y

## 初步建立数据库

```R
require(rWikiPathways)
wp <- downloadPathwayArchive(organism='Homo sapiens', format='gmt')
text <- readLines(wp, encoding = "UTF-8")
text <- strsplit(text, "\t")
WP <- list(TERM2GENE=data.frame(), TERM2NAME=data.frame())
for (i in 1:length(text)){
    line <- text[[i]]
    gsid <- line[1]
    WP$TERM2NAME <- rbind(WP$TERM2NAME, c(i,gsid))
    for(k in 3:length(line)){
        WP$TERM2GENE <- rbind(WP$TERM2GENE, c(i,line[k]))
    }
}
colnames(WP$TERM2NAME) <- c('gsid', 'name')
colnames(WP$TERM2GENE) <- c('gsid', 'gene')
tmp <- AnnotationDbi::select(org.Hs.eg.db::org.Hs.eg.db,keys=WP$TERM2GENE$gene,columns='SYMBOL', keytype='ENTREZID')
```

## 修订部分更新

*   tmp\[is.na(tmp$SYMBOL),\] 找到所有转换失败的ENTREZID，比如 388813
*   访问 https://www.ncbi.nlm.nih.gov/gene/<ENTREZID> 得到更新信息
*   比如 388813 现在已经更名为 64092，HGNC确认其官方SYMBOL为 SAMSN1
*   tmp\[is.na(tmp$SYMBOL), 'SYMBOL'\] <- c('SAMSN1', rep('S1PR3',3))，手动更新信息
*   WP$TERM2GENE$gene <- tmp$SYMBOL 更改ENTREZID为SYMBOL
*   saveRDS(WP,'WP.hsa.rds') 保存数据库
*   自定义数据库的使用方法见下面的博文

https://occdn.limour.top/2128.html