---
title: clusterProfiler：自定义数据库
tags: []
id: '1923'
categories:
  - - uncategorized
date: 2022-07-21 00:07:18
---

clusterProfiler中提供了enricher和GSEA两个函数，enricher包装了**[Over-representation analysis](https://zhuanlan.zhihu.com/p/453991735)**，GSEA包装了[Gene Set Enrichment Analysis](https://www.jianshu.com/p/48d1e043fb63)。这两个函数与其他的enrichKEGG、gseKEGG的唯一不同点是提供的不是KEGG数据库，而是TERM2GENE和TERM2NAME两个参数。

从[gson格式初探](https://occdn.limour.top/2126.html)中，我们可以得到一个Anno对象kk，恰好其中就有kk@gsid2gene、kk@gsid2name两个属性，因此我们可以得知gsid2gene、gsid2name都是一个两列的长数据框：gsid2gene的第一列是**gsid**、表示通路ID，第二列是**gene**、表示通路基因；gsid2name的第一列是**gsid**，第二列是****name****、对通路的描述。有了这些知识，我们可以尝试自己构建一个数据库。

## 构建尝试

包自带的KEGG数据分析起来需要输入ENTREZID。而常规的GTF文件中并不包含这个，通过[对比](https://occdn.limour.top/1934.html)得到的结果只有SYMBOL和ENSEMBL，因此直接转换难免出现不能对应的情况。所以我们需要一个以SYMBOL为**gene**的gsid2gene数据库。

```R
kk_SYMBOL <- list()
kk_SYMBOL[['TERM2NAME']] <- kk@gsid2name
kk_SYMBOL[['TERM2GENE']] <- kk@gsid2gene
tmp <- AnnotationDbi::select(org.Hs.eg.db::org.Hs.eg.db,keys=kk_SYMBOL$TERM2GENE$gene,columns='SYMBOL', keytype='ENTREZID')
kk_SYMBOL$TERM2GENE$gene <- tmp$SYMBOL
kk_SYMBOL$TERM2GENE
```

通过上面的代码，我们得到了一个**gene**为SYMBOL，其他与KEGG完全一致的新数据库，让我们放入enricher中进行测试，看看是否能行。

```R
kegg.d <- enricher(gene = subset(rres, log2FoldChange < -0.1 & padj < 0.05)$symbol,
      universe = rres$symbol,
      pAdjustMethod = "fdr",
      qvalueCutoff = 0.05,
      TERM2GENE = kk_SYMBOL$TERM2GENE,
      TERM2NAME = kk_SYMBOL$TERM2NAME)
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
f_kegg_p(kegg.d, n =15) %>% f_title("kegg.d")
require(enrichplot)
# options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
# BiocManager::install("ggnewscale")
kegg.d2 <- pairwise_termsim(kegg.d)
emapplot(kegg.d2,showCategory=15) 
rres <- rres[order(rres$log2FoldChange, decreasing = T),]
lgene <- rres$log2FoldChange
names(lgene) <- rres$symbol
options(repr.plot.width=8, repr.plot.height=8)
cnetplot(kegg.d2, foldChange=lgene)
```

![](https://img.limour.top/archives_2023/2022/07/21/62d88e9365f97.webp)

显然，结果是成功的！

## 豁然开朗

既然可行，那我们来构建[HALLMARKS](http://www.gsea-msigdb.org/gsea/downloads.jsp)试试。

*   下载 [h.all.v7.5.1.symbols.gmt](http://www.gsea-msigdb.org/gsea/msigdb/download_file.jsp?filePath=/msigdb/release/7.5.1/h.all.v7.5.1.symbols.gmt)，上传到服务器上
*   GMT格式为制表符分隔文件，第一列是通路名称，第二列是描述，第三列以及之后是对应通路下的基因，每个通路为一行。
*   使用下面的代码构建我们自己的clusterProfiler数据库

```R
text <- readLines("~/upload/zl_liu/gsea/h.all.v7.5.1.symbols.gmt", encoding = "UTF-8")
text <- strsplit(text, "\t")
H.ALL <- list(TERM2GENE=data.frame(), TERM2NAME=data.frame())
for (i in 1:length(text)){
    line <- text[[i]]
    gsid <- line[1]
    H.ALL$TERM2NAME <- rbind(H.ALL$TERM2NAME, c(i,gsid))
    for(k in 3:length(line)){
        H.ALL$TERM2GENE <- rbind(H.ALL$TERM2GENE, c(i,line[k]))
    }
}
colnames(H.ALL$TERM2NAME) <- c('gsid', 'name')
colnames(H.ALL$TERM2GENE) <- c('gsid', 'gene')
saveRDS(H.ALL, 'HALLMARKS.rds')
```

得到自己的HALLMARKS后，使用前面的方法即可进行enricher和GSEA分析。

```R
H.ALL <- readRDS('HALLMARKS.rds')
gse.H <- GSEA(gene = lgene,
             pAdjustMethod = "fdr",
             eps = 0,
             TERM2GENE = H.ALL$TERM2GENE,
             TERM2NAME = H.ALL$TERM2NAME)
gseaplot2(gse.H,geneSetID=head(which(gse.H@result$enrichmentScore < -0.3),6))
```

![](https://img.limour.top/archives_2023/2022/07/21/62d8989995e3f.webp)