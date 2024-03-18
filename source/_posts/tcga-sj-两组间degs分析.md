---
title: TCGA (三) 两组间DEGs分析
tags: []
id: '892'
categories:
  - - 生物信息学
  - - 组织测序分析
date: 2021-10-05 06:37:03
---

## 第一步 读取数据

*   TCGA

```
library(TCGAbiolinks)
library(plyr)
library(limma)
library(biomaRt)
library(SummarizedExperiment)

# 配置数据路径
root_path = "~/zlliu/R_data/TCGA"
 
# 配置结果保存路径
output_path = root_path
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

# 一般的前列腺癌 GDC Data Portal 是 hg38 的
PRAD <- GDCquery(project = 'TCGA-PRAD',
                  data.category = "Transcriptome Profiling",
                  data.type = "Gene Expression Quantification", 
                  workflow.type = "HTSeq - Counts")
# mCRPC
mCRPC <- GDCquery(project = 'WCDT-MCRPC',
              data.category = "Transcriptome Profiling",
              data.type = "Gene Expression Quantification", 
              workflow.type = "HTSeq - Counts")
 
# 选择病例列 ，不加cols参数则是完整结果的全部列
PRAD_cases <- getResults(PRAD,cols=c("cases"))
mCRPC_cases <- getResults(mCRPC,cols=c("cases"))
 
# 选择癌组织数据
PRAD_tp  <- TCGAquery_SampleTypes(barcode = PRAD_cases, typesample = "TP")
 
PRAD_D <- GDCquery(project = 'TCGA-PRAD',
                  data.category = "Transcriptome Profiling",
                  data.type = "Gene Expression Quantification", 
                  workflow.type = "HTSeq - Counts",
                  barcode = PRAD_tp)
mCRPC_D <- GDCquery(project = 'WCDT-MCRPC',
              data.category = "Transcriptome Profiling",
              data.type = "Gene Expression Quantification", 
              workflow.type = "HTSeq - Counts",
              sample.type = "Metastatic", #下载癌组织的数据
              barcode = mCRPC_cases)
saveRDS(PRAD_D,"PRAD_D.rds")
saveRDS(mCRPC_D,"mCRPC_D.rds")
 
GDCdownload(query = PRAD_D)
GDCdownload(query = mCRPC_D)
 
PRAD_pre1 <- GDCprepare(query = PRAD_D, save = TRUE, save.filename = "PRAD_pre1.rda")
mCRPC_pre1 <- GDCprepare(query = mCRPC_D, save = TRUE, save.filename = "mCRPC_pre1.rda")
 
# 导出counts矩阵
# PRAD_pre <- PRAD_pre1@assays@data$`HTSeq - Counts`
# colnames(PRAD_pre) <- PRAD_pre1@colData@rownames

 
# mCRPC_pre <- mCRPC_pre1@assays@data$`HTSeq - Counts`
# colnames(mCRPC_pre) <- mCRPC_pre1@colData@rownames

# library(Matrix)
# library(Seurat)
# library(plyr)
# library(dplyr)
# library(patchwork)
# library(purrr)

```

*   GEO

```
f_intersect <- function(lc_ol, t_low=1){
    com_gene <- Reduce(intersect,lapply(lc_ol, rownames))
    com_gene <- com_gene[com_gene != ""]
    res = NULL
    res_g <- NULL
    for(i in 1:length(lc_ol)){
        res <- cbind(res, lc_ol[[i]][com_gene,])
        res_g  <- c(res_g, rep(names(lc_ol)[i], ncol(lc_ol[[i]])))
    }
    res <- res[rowMeans(res) > t_low,]
    list(c=res, g=res_g)
}

f_intersect_mg <- function(lc_glN, lc_gln){
    res <- NULL
    for (i in 1:length(lc_glN)){
        res  <- c(res, rep(lc_glN[i], lc_gln[i]))
    }
    res
}

f_intersect_g <- function(lc_counts, lc_ctN, lc_ctn, lc_trN){
    c(rep(lc_ctN, lc_ctn), rep(lc_trN, ncol(lc_counts)-lc_ctn))
}

f_intersect_c <- function(lc_g){
    paste(lc_g,1:length(lc_g),sep='')
}

f_intersect_r <- function(lc_d, lc_s, lc_e, lc_sub=F){
    lc_s = which(colnames(lc_d) == lc_s)
    lc_e = which(colnames(lc_d) == lc_e)
    if(lc_sub){
        return(lc_d[,-(lc_s:lc_e)])
    }else{
        return(lc_d[,(lc_s:lc_e)])
    }
}

```

```
f_id2name <- function(lc_exp, lc_db){
    if(!is.data.frame(lc_db)){
        lc_ids <- toTable(lc_db)
    }else{
        lc_ids <- lc_db
    }
    res <- lc_exp[rownames(lc_exp) %in% lc_ids[[1]],]
    lc_ids=lc_ids[match(rownames(res),lc_ids[[1]]),]
    lc_tmp = by(res,
         lc_ids[[2]],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res = res[rownames(res) %in% lc_probes,]
    rownames(res)=lc_ids[match(rownames(res),lc_ids[[1]]),2]
    res
}

library(GEOquery)
f_getGPL <- function(lc_GPLN, lc_local = F){
    options(stringsAsFactors = F)
    if (!file.exists(lc_GPLN)){dir.create(lc_GPLN)}
    if(lc_local){
        gpl=read.table(file.path(lc_GPLN, lc_GPLN),
               header = TRUE,fill = T,sep = "\t",
               comment.char = "#",
               stringsAsFactors = FALSE,
               quote = "")
        return(gpl)
    }else{
        return(Table(getGEO(lc_GPLN,destdir = lc_GPLN)))
    }
}

f_getGEO <- function(lc_GEOID, lc_GPLN){
    if (!file.exists(lc_GEOID)){dir.create(lc_GEOID)}
    gset <- getGEO(lc_GEOID, GSEMatrix =TRUE, AnnotGPL=F, destdir=lc_GEOID)
    if (length(gset) > 1) idx <- grep(lc_GPLN, attr(gset, "names")) else idx <- 1
    gset <- gset[[idx]]
    fvarLabels(gset) <- make.names(fvarLabels(gset))
    exprs(gset)
}

```

```
GSE70768 <- f_getGEO("GSE70768", "GPL10558")
GPL10558 = f_getGPL('GPL10558')[,c('ID','Symbol')]
GSE70768 <- f_id2name(GSE70768, GPL10558)
# GSE70768 <- 2^GSE70768 # log2 transformed and quantile normalized
# GSE70768_CRPC <-  CreateSeuratObject(counts = GSE70768, project = "GSE70768_CRPC") 
# GSE70768_CRPC[['group']] <- 'CRPC'

GPL15659 <- f_getGPL('GPL15659')[,c('ID','GENE_SYMBOL')]
GSE74367 <-  f_getGEO("GSE74367", "GPL15659")
GSE74367 <- f_id2name(GSE74367, GPL15659)
# GSE74367 <- 2^GSE74367 # batch-normalized log2 Cy3 signal intensity
# GSE74367_CRPC <-  CreateSeuratObject(counts = GSE74367, project = "GSE74367_CRPC") 
# GSE74367_CRPC[['group']] <- 'CRPC'

GSE101607 <- f_getGEO("GSE101607", "GPL10558")
GSE101607 <- f_id2name(GSE101607, GPL10558)
# GSE101607_CRPC <-  CreateSeuratObject(counts = GSE101607, project = "GSE101607_CRPC") 
# GSE101607_CRPC[['group']] <- 'CRPC'

```

## 第二步 整合数据

```
f_tcga_fpkm <- function(lc_pre1, lc_exons_gene_lens){
    lc_pre1_l <- lc_exons_gene_lens[lc_pre1@rowRanges$ensembl_gene_id,]/1000 # 转kb
    lc_pre1_rpk <- lc_pre1@assays@data$`HTSeq - Counts`/lc_pre1_l
    lc_pre1_fpkm <- t(t(lc_pre1_rpk)/colSums(lc_pre1@assays@data$`HTSeq - Counts`) * 10^6)
    lc_pre1_fpkm
    colnames(lc_pre1_fpkm) <- lc_pre1@colData@rownames
    rownames(lc_pre1_fpkm) <- lc_pre1@rowRanges$external_gene_name
    lc_pre1_fpkm
}
f_tcga_row <- function(lc_pre1){
    lc_counts <- lc_pre1@assays@data$`HTSeq - Counts`
    colnames(lc_counts) <- lc_pre1@colData@rownames
    rownames(lc_counts) <- lc_pre1@rowRanges$external_gene_name
    lc_counts
}
fpkmToTpm <- function(fpkm){
    exp(log(fpkm) - log(sum(fpkm)) + log(1e6))
}
f_fpkmToTpm <- function(l_e){
     apply(l_e,2,fpkmToTpm)
}
f_tcga_getSampleID <- function(lc_pre1){
    cn <- lc_pre1@colData@rownames
    cn <- unlist(t(data.frame(strsplit(a, "-"))[4,])[,1])
    names(cn) <- NULL
    substr(cn,1,2)
}
```

```
# https://api.gdc.cancer.gov/data/25aa497c-e615-4cb7-8751-71f744f9691f
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-1.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-1.webp)

下载解压

```
library(GenomicFeatures)
txdb <- makeTxDbFromGFF("gencode.v22.annotation.gtf",format="gtf")
exons_gene <- exonsBy(txdb, by = "gene")
exons_gene_lens <- lapply(exons_gene,function(x){sum(width(reduce(x)))})
exons_gene_lens1 <- as.data.frame(exons_gene_lens)
exons_gene_lens1 <- t(exons_gene_lens1)
exons_gene_lens1 <- as.data.frame(exons_gene_lens1)
rownames(exons_gene_lens1) <- gsub("\\.(\\.?\\d*)","",rownames(exons_gene_lens1))
saveRDS(exons_gene_lens1,'exons_gene_lens.rds')
```

```
sym2id.db <- data.frame(sym=PRAD_pre1@rowRanges$external_gene_name, ID=PRAD_pre1@rowRanges$ensembl_gene_id)
saveRDS(sym2id.db,'sym2id.db')
f_GEO2fpkm <- function(lc_exp, lc_exons_gene_lens){
        lc_pre1_l <- lc_exons_gene_lens[rownames(lc_exp),]/1000 # 转kb
        lc_pre1_rpk <-lc_exp/lc_pre1_l
        lc_pre1_fpkm <- t(t(lc_pre1_rpk)/colSums(lc_exp) * 10^6)
        lc_pre1_fpkm
}
```

```
# PRAD_pre <- f_intersect_sub(PRAD_pre, c('TCGA-HC-8265-01B-04R-2302-07','TCGA-HC-8258-01B-05R-2302-07', 'TCGA-HC-7740-01B-04R-2302-07'))
GSE101607 <- f_log2(GSE101607)
```

```
tcga <- f_intersect(list(tcga_PRAD=f_tcga_row(PRAD_pre1), tcga_mCRPC=f_tcga_row(mCRPC_pre1)))

GSE70768_CRPC <- GSE70768[,1:13]
GSE70768_PRAD <- f_intersect_r(GSE70768, 'GSM1817720', 'GSM1817832')

GSE74367_CRPC <- GSE74367[,21:65]
GSE74367_PRAD <- f_intersect_r(GSE74367, 'GSM1918981', 'GSM1918991')

GSE101607_CRPC <- GSE101607[,1:40]
GSE101607_PRAD1 <- f_intersect_r(GSE101607, 'GSM2710899', 'GSM2710905')
GSE101607_PRAD2 <- GSE101607[, 'GSM2710886']
GSE101607_PRAD <- cbind(GSE101607_PRAD2, GSE101607_PRAD1)
colnames(GSE101607_PRAD)[[1]] <- 'GSM2710886'

GSE28403_CRPC <- f_intersect_sub(GSE28403, c('GSM701959','GSM701961', 'GSM701962', 'GSM701963'))
GSE28403_PRAD <- GSE28403[, c('GSM701959','GSM701961', 'GSM701962', 'GSM701963')]

```

## 第三步 批次效应

```
f_pca <- function(lc_counts, lc_g){
    res <- prcomp(lc_counts)
    res <- data.frame(res$rotation, g = lc_g)
    res$CB <- rownames(res)
    res
}

library(ggplot2)
library(tidyverse)
f_pca_p <- function(lc_pca, x='PC1', y='PC2'){
    ggplot(lc_pca,aes(x=!!sym(x),y=!!sym(y),color=g))+
    geom_point()+
    theme_bw()+
    theme(panel.border=element_blank(),panel.grid.major=element_blank(),
          panel.grid.minor=element_blank(),axis.line= element_line(colour = "blue"))
}

library(umap)
f_umap <- function(lc_pca,dims=1:30){
    res <- umap(lc_pca[,dims])
    res <- data.frame(res$layout)
    colnames(res) <- c('UMAP1', 'UMAP2')
    res$CB <- rownames(res)
    tp <- lc_pca[,c('g','CB')]
    res <- merge(res, tp, by="CB")
    res
}
f_umap_p <- function(lc_umap){
    ggplot(lc_umap,aes(x=UMAP1,y=UMAP2,color=g))+
    geom_point()+
    theme_bw()+
    theme(panel.border=element_blank(),panel.grid.major=element_blank(),
          panel.grid.minor=element_blank(),axis.line= element_line(colour = "blue"))
}

library(ggplot2)
library(reshape2)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)
 
f_gl_boxp <- function(lc_exp, lc_g, lc_delta_ylim){
    lc_exp_L = melt(lc_exp)
    colnames(lc_exp_L) = c('symbol','sample','value')
    lc_exp_L$group = lc_g
    p=ggplot(lc_exp_L,aes(x=sample,y=value,fill=group))+geom_boxplot(outlier.shape = NA)
    p=p+theme_set(theme_set(theme_bw(base_size=20)))
    p=p+theme(text=element_text(face='bold'),axis.text.x=element_text(angle=30,hjust=1),axis.title=element_blank())
    lc_ylim = unlist(by(lc_exp_L,
     lc_exp_L$sample,
     function(x){boxplot.stats(x$value)$stats[c(1, 5)]}))
    lc_ylim <- c(min(lc_ylim), max(lc_ylim)) + lc_delta_ylim
    p=p + coord_cartesian(ylim = lc_ylim)
    p
}
 
f_icg_boxp <- function(lc_exp, lc_icg, lc_g){
    lc_exp_L = melt(lc_exp[lc_icg,])
    lc_exp_L <- cbind(lc_exp_L, rownames(lc_exp_L))
    colnames(lc_exp_L)=c('value','sample')
    lc_exp_L$group = lc_g
    p=ggplot(lc_exp_L,aes(x=group,y=value,fill=group))+geom_boxplot()
    p=p+stat_summary(fun="mean",geom="point",shape=23,size=3,fill="red")
    p=p+theme_set(theme_set(theme_bw(base_size=20)))
    p=p+theme(text=element_text(face='bold'),axis.text.x=element_text(angle=30,hjust=1),axis.title=element_blank())
    p=p+ggtitle(lc_icg)+theme(plot.title = element_text(hjust = 0.5))
    p
}

# 若缺少sva包，则安装
# options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
# BiocManager::install("sva", update=F)
library(sva)
library(limma)
f_removeBE <- function(lc_dat, lc_batch, lc_g, use_ComBat=F){
    if(use_ComBat){
        lc_mod <- model.matrix(~as.factor(lc_g))
        res <- ComBat(dat = lc_dat, batch = as.factor(lc_batch), mod = lc_mod)
    }else{
        lc_mod <- model.matrix(~0+as.factor(lc_g))
        res <- removeBatchEffect(lc_dat, batch = as.factor(lc_batch), design = lc_mod)
    }
    res
}

library(ggdendro)
f_DEG_hclust <- function(lc_counts){
    ggdendrogram(hclust(dist(t(lc_counts))), rotate = T, size = 3)+theme(axis.text = element_text(size=14,face = "bold"))
}

```

```
gse$m <- f_removeBE(gse$c, gse$b, gse$t, use_ComBat = T)
f_icg_boxp(gse$m, 'GAPDH', gse$t)
f_icg_boxp(normalizeBetweenArrays(cbind(GSE74367_PRAD,GSE74367_CRPC)),'GAPDH',
           f_intersect_mg(c('PRAD', 'CRPC'),c(ncol(GSE74367_PRAD),ncol(GSE74367_CRPC))))
f_icg_boxp(normalizeBetweenArrays(gse$m), 'GAPDH', gse$t)
options(repr.plot.width=10, repr.plot.height=10)
(f_icg_boxp(gse$c, 'GAPDH', gse$g) + f_icg_boxp(gse$m, 'GAPDH', gse$g))/
(f_icg_boxp(gse$c, 'GAPDH', gse$t) + f_icg_boxp(gse$m, 'GAPDH', gse$t))
(f_pca_p(f_pca(gse$c,gse$g)) + f_pca_p(f_pca(gse$m,gse$g)))/
(f_pca_p(f_pca(gse$c,gse$t)) + f_pca_p(f_pca(gse$m,gse$t)))
(f_umap_p(f_umap(f_pca(gse$c,gse$g))) + f_umap_p(f_umap(f_pca(gse$m,gse$g))))/
(f_umap_p(f_umap(f_pca(gse$c,gse$t))) + f_umap_p(f_umap(f_pca(gse$m,gse$t))))
```

## 第四步 计算DEGs

```
library(limma)
library(pheatmap)
f_getDEG <- function(lc_counts, lc_group_list, lc_contrasts){
    if (length(lc_contrasts) == 2){lc_contrasts = paste0(lc_contrasts,collapse = "-")}
    lc_design <- model.matrix(~0+factor(lc_group_list))
    colnames(lc_design) <- levels(factor(lc_group_list))
    rownames(lc_design) <- colnames(lc_counts)
    lc_fit <- lmFit(lc_counts, lc_design)
    lc_cont.matrix <- makeContrasts(contrasts=lc_contrasts, levels=lc_design)
    print(lc_cont.matrix)
    lc_fit2 <- contrasts.fit(lc_fit, lc_cont.matrix)
    lc_fit2 <- eBayes(lc_fit2, 0.01)
    topTable(lc_fit2, adjust="fdr", sort.by="B", number=Inf)
}

library(ggplot2)
library(ggrepel)
f_DEG_Volcano <- function(lc_logFC, lc_p, lc_gene, Threshold_logFC = 1, Threshold_p = 0.05, lc_rep=1:10, lc_p_m=1e-10){
    col_vector = rep(rgb(108, 200, 228, maxColorValue = 255), length(lc_logFC))
    col_vector[lc_p < Threshold_p & lc_logFC > Threshold_logFC] = rgb(226, 61, 75, maxColorValue = 255)
    col_vector[lc_p < Threshold_p & lc_logFC < -Threshold_logFC] = rgb(232, 168, 71, maxColorValue = 255)
    lc_p[lc_p < lc_p_m] = lc_p_m
    lc_p[lc_p > 1  is.na(lc_p)] = 1
    df = data.frame(logFC <- lc_logFC, `-log10(P)` <- -log10(lc_p), col <- col_vector, gene <- lc_gene)
    colnames(df) <- c('logFC', '-log10(P)', "col", "gene")
    lc_tp_logFC <- df$logFC
    lc_tp_logFC[lc_p>=Threshold_p] = 0
    lc_idx <- order(lc_tp_logFC)[c(lc_rep, length(lc_gene)+1-lc_rep)]
    res <- ggplot() + geom_point(aes(logFC, `-log10(P)`, col=I(col)), data = df) 
    res <- res + theme_bw() + theme(panel.grid=element_line(colour=NA))
    res <- res + geom_hline(yintercept=-log10(Threshold_p), linetype="longdash")
    res <- res + geom_vline(xintercept=c(Threshold_logFC, -Threshold_logFC), linetype="longdash")
    res <- res + geom_text_repel(data=df[lc_idx,],aes(logFC,`-log10(P)`,label=gene), force=T, max.overlaps=Inf)
    res
}

```

```
library(DESeq2)
f_dds <- function(l_counts, lc_g, lc_batch=NULL, t_low=1){
    lc_cond <- factor(lc_g)
    lc_coldata = data.frame(row.names=colnames(l_counts), lc_cond)
    if(length(lc_batch) != 0 ){
        lc_bt <- factor(lc_batch)
        lc_dds <- DESeqDataSetFromMatrix(countData = l_counts, colData = lc_coldata, design= ~lc_bt + lc_cond)
    }else{
        lc_dds <- DESeqDataSetFromMatrix(countData = l_counts, colData = lc_coldata, design= ~lc_cond)
    }
    lc_dds <- lc_dds[rowSums(counts(lc_dds))>t_low, ]
    dds_out <- DESeq(lc_dds)
    res <- results(dds_out)
    data.frame(res)
}
```

```
options(repr.plot.width=10, repr.plot.height=10)
gse$deg <- f_getDEG(gse$m,gse$t,c('CRPC','PRAD'))
f_DEG_Volcano(gse$deg$logFC, gse$deg$adj.P.Val, rownames(gse$deg), Threshold_logFC = 0.5, lc_p_m = 1e-30)
```

```
tcga$c <- tcga$c[rownames(tcga$c) %in% rownames(gse$m),]
tcga$m <- normalizeBetweenArrays(tcga$c)
options(repr.plot.width=10, repr.plot.height=15)
(f_icg_boxp(tcga$c, 'GAPDH', tcga$g) + f_icg_boxp(tcga$m, 'GAPDH', tcga$g))/
(f_pca_p(f_pca(tcga$c,tcga$g)) + f_pca_p(f_pca(tcga$m,tcga$g)))/
(f_umap_p(f_umap(f_pca(tcga$c,tcga$g))) + f_umap_p(f_umap(f_pca(tcga$m,tcga$g))))
tcga$pca <- f_pca(tcga$m,tcga$g)
tcga$t <- tcga$g
names(tcga$t) <- colnames(tcga$c)
tcga$pca <- subset(tcga$pca, (PC1 > 0.04  PC1 < 0.03)&(PC2 > 0.06  PC2 < 0.01))
tcga$t <- tcga$t[rownames(tcga$pca)]
table(tcga$t)
tcga$m <- tcga$m[,rownames(tcga$pca)]
options(repr.plot.width=10, repr.plot.height=15)
(f_icg_boxp(tcga$c, 'GAPDH', tcga$g) + f_icg_boxp(tcga$m, 'GAPDH', tcga$t))/
(f_pca_p(f_pca(tcga$c,tcga$g)) + f_pca_p(f_pca(tcga$m,tcga$t)))/
(f_umap_p(f_umap(f_pca(tcga$c,tcga$g))) + f_umap_p(f_umap(f_pca(tcga$m,tcga$t))))
tcga$pca <- f_pca(tcga$m,tcga$t)
tcga$pca <- subset(tcga$pca, (PC1 > -0.03 & PC2 >0.05)  (PC1 < -0.04 & PC2 < 0.03))
tcga$t <- tcga$t[rownames(tcga$pca)]
table(tcga$t)
tcga$m <- tcga$m[,rownames(tcga$pca)]
options(repr.plot.width=10, repr.plot.height=15)
(f_icg_boxp(tcga$c, 'GAPDH', tcga$g) + f_icg_boxp(tcga$m, 'GAPDH', tcga$t))/
(f_pca_p(f_pca(tcga$c,tcga$g)) + f_pca_p(f_pca(tcga$m,tcga$t)))/
(f_umap_p(f_umap(f_pca(tcga$c,tcga$g))) + f_umap_p(f_umap(f_pca(tcga$m,tcga$t))))
names(tcga$g) <- colnames(tcga$c)
tcga$c <- tcga$c[,rownames(tcga$pca)]
tcga$g <- tcga$g[rownames(tcga$pca)]
tcga$deg <- f_dds(tcga$c, tcga$g, t_low=10)
tcga$logFC_cutoff <- mean(abs(tcga$deg$log2FoldChange)) + 2*sd(abs(tcga$deg$log2FoldChange))
f_DEG_Volcano(tcga$deg$log2FoldChange, tcga$deg$padj, rownames(tcga$deg), Threshold_logFC = mean(abs(tcga$deg$log2FoldChange)) + 2*sd(abs(tcga$deg$log2FoldChange)), lc_p_m = 1e-300)
```