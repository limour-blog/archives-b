---
title: ReactomePA试用
tags: []
id: '2001'
categories:
  - - uncategorized
date: 2022-09-02 14:00:17
---

## 安装补充包

*   [conda activate clusterprofiler](https://occdn.limour.top/2275.html)
*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://github.com/YuLab-SMU/ReactomePA/archive/refs/heads/master.zip -O ReactomePA-master.zip
*   BiocManager::install("reactome.db")
*   devtools::install\_local('ReactomePA-master.zip')

## 读入数据

[差异基因分析](https://occdn.limour.top/2132.html)

```R
library(stringr)
library(org.Hs.eg.db)
DEG <- subset(readRDS('DEG.rds'), !grepl('pseudogene', gene_type) & baseMean > quantile(baseMean)['25%'] & padj < 0.05)
rownames(DEG) <- t(as.data.frame(str_split(DEG$gene_id, '\\.')))[,1]
allEntrez = clusterProfiler::bitr(rownames(DEG), fromType="ENSEMBL", toType="ENTREZID", OrgDb=org.Hs.eg.db)
DEG$ENSEMBL <- rownames(DEG)
lfc <- merge(data.frame(DEG), allEntrez, by="ENSEMBL")
lfc <- lfc[order(lfc$log2FoldChange, decreasing=TRUE),]
geneList <- lfc$log2FoldChange
names(geneList) <- lfc$ENTREZID
```

```R
x <- readRDS('DEG_filer.rds')
rownames(x) <- t(as.data.frame(str_split(x$gene_id, '\\.')))[,1]
cand.entrez = clusterProfiler::bitr(rownames(x), fromType="ENSEMBL", toType="ENTREZID", OrgDb=org.Hs.eg.db)$ENTREZID
```

## 进行分析

### ORA

```R
set.seed(123)
pway = ReactomePA::enrichPathway(gene = cand.entrez)
pway = clusterProfiler::setReadable(pway, OrgDb=org.Hs.eg.db)
pway = enrichplot::pairwise_termsim(pway)
pway@result
```

### GSEA

```R
set.seed(123)
pwayGSE <- ReactomePA::gsePathway(geneList, eps = 0)
pwayGSE = clusterProfiler::setReadable(pwayGSE, OrgDb=org.Hs.eg.db)
pwayGSE = enrichplot::pairwise_termsim(pwayGSE)
pwayGSE@result
```