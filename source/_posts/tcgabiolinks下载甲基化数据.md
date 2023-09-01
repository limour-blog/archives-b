---
title: TCGAbiolinks下载甲基化数据
tags: []
id: '2010'
categories:
  - - uncategorized
date: 2022-09-08 16:32:59
---

## 安装补充包

*   [conda activate tcga](https://occdn.limour.top/1655.html)
*   \# conda install -c bioconda bioconductor-sesamedata -y
*   conda install -c conda-forge r-magick -y
*   BiocManager::install("sesameData")
*   BiocManager::install("sesame")
*   BiocManager::install("doParallel")
*   BiocManager::install("ComplexHeatmap")
*   BiocManager::install("matlab")
*   conda install -c bioconda bioconductor-motifstack -y
*   \# conda install -c bioconda bioconductor-bsgenome.hsapiens.ucsc.hg38 -y
*   BiocManager::install("BSgenome.Hsapiens.UCSC.hg38")
*   BiocManager::install("BSgenome.Hsapiens.UCSC.hg19")
*   \# conda install -c bioconda bioconductor-genomicranges -y
*   conda install -c bioconda bioconductor-rgadem -y

## 下载数据

```R
library(TCGAbiolinks)
query_met.hg38 <- GDCquery(
    project= "TCGA-PRAD", 
    data.category = "DNA Methylation", 
    data.type = "Methylation Beta Value",
    platform = "Illumina Human Methylation 450"
)
GDCdownload(query_met.hg38)
data.hg38 <- GDCprepare(query_met.hg38)
saveRDS(data.hg38, 'prad.meth.rds')
```

## 清洗数据

```R
library(SummarizedExperiment)
# 删除包含 NA 值的探针
data.hg38 <- subset(data.hg38,subset = (rowSums(is.na(assay(data.hg38))) == 0))
# 去除重复样本
data.hg38 <- data.hg38[, substr(colnames(data.hg38), 14, 16) == "01A"]
group <- readRDS('../idea_2/fig3.2/fig5/tcga.predict.rds')
gRow <- intersect(data.hg38$patient, rownames(group))
group <- group[gRow,]
data.hg38 <- data.hg38[, substr(colnames(data.hg38), 1, 12) %in% gRow]
data.hg38$group <- group[substr(colnames(data.hg38), 1, 12), 'group']
```

## 计算整体差异

```R
TCGAvisualize_meanMethylation(
  data.hg38,
  groupCol = "group",
  group.legend  = "Groups",
  filename = NULL,
  print.pvalue = TRUE
)
```

## **识别差异甲基化** CpG **位点**

```R
res <- TCGAanalyze_DMC(
  data.hg38,
  # colData 函数获取的矩阵中分组列名
  groupCol = "group",
  group1 = "High Risk",
  group2 = "Low Risk",
  p.cut = 0.05,
  diffmean.cut = 0.15,
  save = FALSE,
  legend = "State",
  plot.filename = "./PRAD_metvolcano.png",
  cores =  8
)
```

### 绘制热图

```R
sig_met <- subset(res, status != "Not Significant")
res_data <- subset(data.hg38, subset = (rownames(data.hg38) %in% rownames(sig_met)))
library(circlize)
library(ComplexHeatmap)
group <- group[substr(colnames(data.hg38), 1, 12), ]
#定义注释信息
ha <- HeatmapAnnotation(`Risk group` = group$group,
                        `Risk Score` = group$Risk_Score,
                        col = list(
                            `Risk group` = c("Low Risk" = "#99CCFFAA", 'High Risk' = '#FF6666AA')
                        ),
#                         annotation_name_gp = gpar(fontsize = 14),
                        show_annotation_name = TRUE)
heatmap  <- Heatmap(
  assay(res_data),
  name = "DNA methylation",
  col = matlab::jet.colors(200),
  show_row_names = FALSE,
  cluster_rows = TRUE,
  cluster_columns = FALSE,
  show_column_names = FALSE,
  bottom_annotation = ha,
  column_title = "DNA Methylation",
  column_order = order(group$Risk_Score)
)
pdf("./prad_meth_heatmap.pdf",width = 6, height = 4);{
    draw(heatmap, annotation_legend_side =  "bottom")
};dev.off()
```

## **模体分析**（需要hg19）

```R
query_met.hg19 <- GDCquery(
    project= "TCGA-PRAD", 
    data.category = "DNA methylation",
    platform = "Illumina Human Methylation 450",
    legacy = TRUE # hg19
)
GDCdownload(query_met.hg19)
data.hg19 <- GDCprepare(query_met.hg19)
saveRDS(data.hg38, 'prad.meth.hg19.rds')
```

```R
library(rGADEM)
library(GenomicRanges)
library(BSgenome.Hsapiens.UCSC.hg19)
library(motifStack)
probes <- rowRanges(res_data)
# 获取差异的探针并设置 200bp 大小的窗口
sequence <- GRanges(
  seqnames = as.character(seqnames(probes)),
  ranges = IRanges(start = start(ranges(probes)) - 100,
          end = start(ranges(probes)) + 100),
  strand = "*"
)
# 识别模体
gadem <- GADEM(sequence, verbose = FALSE, genome = Hsapiens)
```

因为hg19的数据还在下载，更多内容见[官方教程](https://www.bioconductor.org/packages/release/workflows/vignettes/TCGAWorkflow/inst/doc/TCGAWorkflow.html)