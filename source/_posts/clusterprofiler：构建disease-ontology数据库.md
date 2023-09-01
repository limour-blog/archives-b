---
title: clusterProfiler：构建Disease Ontology数据库
tags: []
id: '1945'
categories:
  - - uncategorized
date: 2022-08-02 21:40:39
---

与常用的基因功能注释数据库类似，[Disease Ontolog](https://disease-ontology.org/)通过参照MeSH, ICD等疾病分类标准，对人类的常见疾病与罕见病进行了归纳整理，提供了一个统一的，标准化的疾病分类系统。

## 安装补充包

*   [conda activate clusterprofiler](https://occdn.limour.top/2126.html)
*   conda install -c bioconda bioconductor-biomart -y

## 获得基础数据

```R
tmp <- DOSE:::get_DO_data('DO')
DO <- list(TERM2GENE=data.frame(), TERM2NAME=data.frame())
DO$TERM2NAME <- as.data.frame(tmp$PATHID2NAME)
names(DO$TERM2NAME) <- 'name'
DO$TERM2NAME['gsid'] <- rownames(DO$TERM2NAME)
rownames(DO$TERM2NAME) <- NULL
DO$TERM2NAME <- DO$TERM2NAME[c('gsid', 'name')]
for (PATHID in names(tmp$PATHID2EXTID)){
    line <- tmp$PATHID2EXTID[[PATHID]]
    lc_gsid <- rep(x = PATHID, times = length(line))
    DO$TERM2GENE <- rbind(DO$TERM2GENE, cbind(lc_gsid, line))
}
colnames(DO$TERM2GENE) <- c('gsid', 'gene')
```

## 转化ENTREZID

```R
tmp <- AnnotationDbi::select(org.Hs.eg.db::org.Hs.eg.db,keys=DO$TERM2GENE$gene,columns='SYMBOL', keytype='ENTREZID')
DO$TERM2GENE$gene <- tmp$SYMBOL
DO$TERM2GENE <- na.omit(DO$TERM2GENE)
saveRDS(DO,'DO.hsa.rds')
# tmp[is.na(tmp$SYMBOL),]
# library(biomaRt)
# listMarts() #看看有多少数据库资源
# ensembl=useMart("ensembl")
# listDatasets(ensembl)#看看选择的数据库里面有多少数据表，这个跟物种相关
# ensembl = useDataset("hsapiens_gene_ensembl",mart=ensembl)
# ensembl = useMart("ensembl",dataset="hsapiens_gene_ensembl") # 这是一步法选择人类的ensembl数据库代码
# searchFilters(mart = ensembl, 'entrezid')
# getBM(attributes=c('hgnc_symbol'), filters = 'entrezgene_id', values = c('100128356', '4590'), mart = ensembl)
```