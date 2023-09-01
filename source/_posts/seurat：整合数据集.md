---
title: Seurat (二) 整合数据集
tags: []
id: '645'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-09-19 22:43:37
---

## 第一步 导入程辑包

```
library(Matrix)
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)
```

## 第二步 配置数据路径工作路径

```
# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"

# 配置结果保存路径
output_path = "~/zlliu/R_data/21.07.15.IntegrateData"
if (!file.exists(output_path)){dir.create(output_path)}

```

## 第三步 定义各种ID的转换方法

```
f_id2name <- function(lc_exp, lc_db){
    lc_ids=toTable(lc_db)
    res <- lc_exp[rownames(lc_exp) %in% lc_ids[1],]
    lc_ids=lc_ids[match(rownames(res),lc_ids[1]),]
    lc_tmp = by(res,
         lc_ids[2],
         function(x) rownames(x)[which.max(rowMeans(x))])
    lc_probes = as.character(lc_tmp)
    res = res[rownames(res) %in% lc_probes,]
    rownames(res)=lc_ids[match(rownames(res),lc_ids[1]),2]
    res
}
```

*   lc\_exp：<data.frame>表达矩阵，行名为细胞barcode等区分细胞的标识，列名为转换前的基因标识
*   lc\_db：<data.frame>基因映射表，第一列为转换前的基因标识，第二列为转换后的基因标识

## 第四步 读入数据并转成SeuratObject

### 第一类 table数据的读入

#### 示例一 GRCh38\_ENSG00000223972标识基因的数据

```
gl_id2name <- read.table(paste(root_path, "/ENSG_gene_1", ".txt", sep=""),header=T,na.strings = c("NA")) # 基因映射表


f_sns <- function(lc_data_name, lc_barcode_region){
    sn <- read.table(paste(root_path, lc_data_name, ".tsv", sep=""), header=T, row.names=1) # barcode前16nt为组织标识
    sn <- f_Id2name(sn, gl_id2name)
    sn_br <- read.csv(file.path(root_path, "Split_Seq_FourSamples", "barcode_region",lc_barcode_region), header = F)
    sn_br[1:20,2] <- gsub("BA2/1/3", "BA213", sn_br[1:20,2])
    sn_br[1:20,2] <- gsub("BA4", "BA04", sn_br[1:20,2])
    sn_br[1:20,2] <- gsub("BA6", "BA06", sn_br[1:20,2]) 
    sn_br[1:20,2] <- gsub("BA9", "BA09", sn_br[1:20,2])
    rownames(sn_br) <- sn_br[,1]
    print(sn_br)
    lc_tp_r <- sn_br[substr(colnames(sn),1,16),2]
    sns <- list()
    for(lc_ba in unique(lc_tp_r)){
        sns[[lc_ba]] <- sn[,which(lc_tp_r == lc_ba)]
    }
    sns
}

s09s <- f_sns("/Split_Seq_FourSamples/S9_counts_GRCh38_filtered", "barcode_region_S9.csv")
names(s09s)

s11s <- f_sns("/Split_Seq_FourSamples/S11_counts_GRCh38_filtered", "barcode_region_S11.csv")
names(s11s)

s12s <- f_sns("/Split_Seq_FourSamples/S12_counts_GRCh38_filtered", "barcode_region_S12.csv")
names(s12s)

```

#### 示例二 GEO数据

```
library(GEOquery)
library(limma)
library(pheatmap)


# 自己制作探针注释的方法： https://mp.weixin.qq.com/s/mrtjpN8yDKUdCSvSUuUwcA
# 使用Bioconductor的现成注释：https://blog.csdn.net/weixin_40739969/article/details/103186027
library("illuminaMousev2.db")

f_groupRename <- function(lc_exp, lc_c, lc_group){
    res <- lc_exp[,lc_c]
    colnames(res) <- paste(lc_group,1:length(lc_group),sep='')
    res
}

gset <- getGEO("GSE83472", GSEMatrix =TRUE, AnnotGPL=TRUE)
if (length(gset) > 1) idx <- grep("GPL6885", attr(gset, "names")) else idx <- 1
gset <- gset[[idx]]

library(stringr)
f_getGroup <- function(lc_title, lc_treat, lc_contorl){
    tp_tr <- str_detect(lc_title, lc_treat)
    tp_co <- str_detect(lc_title, lc_contorl)
    lc_ix <- c(which(tp_tr), which(tp_co))
    lc_gl <- c(rep(lc_treat,times=sum(tp_tr)),rep(lc_contorl,times=sum(tp_co)))
    list(gl = lc_gl, ix = lc_ix)
}

gl_group_list <- f_getGroup(pData(gset)$title, 'IR', 'Sham')

gl_counts <- exprs(gset)

gl_counts = f_groupRename(gl_counts,gl_group_list$ix,gl_group_list$gl)

tp_counts <- f_id2name(gl_counts, illuminaMousev2SYMBOL)

```

### 第二类 10x数据

```
# 5、读取全部samples
tp_samples <- list.files(file.path(root_path, "10x"))
tp_dir <- file.path(root_path, "10x", tp_samples)
names(tp_dir) <- tp_samples
counts <- Read10X(data.dir = tp_dir)
scRNA <- CreateSeuratObject(counts, project = 'hBLA',
                            min.cells = 3, min.features = 200)
rm(counts)
rm(tp_dir)
gc()
```

## 第五步 分别质控

```
# 6、QC
scRNA[["percent.mt"]] <- PercentageFeatureSet(scRNA, pattern = "^MT-")

# 6.2、分割数据以便QC
x10s <- SplitObject(scRNA, split.by = 'ident')
rm(scRNA)
gc()

f_sn2so <- function(sn){
    # 以s09s为准
    for (lc_ba in names(s09s)){
        sn[[lc_ba]] <- Matrix(as.matrix(sn[[lc_ba]]))
        sn[[lc_ba]] <- CreateSeuratObject(counts = sn[[lc_ba]], project = lc_ba, min.cells = 3, min.features = 200)
    }
    sn
}

s09s <- f_sn2so(s09s)
s11s <- f_sn2so(s11s)
s12s <- f_sn2so(s12s)
s13s <- f_sn2so(s13s)

s12s[['mm']] <- NULL
s13s[['mm']] <- NULL

x10s
s09s
s11s
s12s
s13s

# 6.3、分别对各样本进行QC
x10s$BA213 <- subset(x10s$BA213,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 4000 & percent.mt < 10)
x10s$BA04 <- subset(x10s$BA04,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 3000 & percent.mt < 10)
x10s$BA09 <- subset(x10s$BA09,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 3500 & percent.mt < 10)
x10s$BA21 <- subset(x10s$BA21,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 4500 & percent.mt < 10)
x10s$BA22 <- subset(x10s$BA22,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 3000 & percent.mt < 10)
x10s$BA39 <- subset(x10s$BA39,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 4000 & percent.mt < 10)
x10s$BA40 <- subset(x10s$BA40,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 4000 & percent.mt < 10)
x10s$BA44 <- subset(x10s$BA44,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 4000 & percent.mt < 10)
x10s$BA45 <- subset(x10s$BA45,
                  subset = nFeature_RNA > 200 & nFeature_RNA < 5000 & percent.mt < 10)

# 6.4、查看QC效果
f_show_QC <- function(sn){
    tp_mergedata <- merge(sn$BA213, y=c(sn$BA04, sn$BA06, sn$BA09, sn$BA21,
                                       sn$BA22, sn$BA37, sn$BA39, sn$BA40,
                                       sn$BA41, sn$BA42, sn$BA44, sn$BA45))
    VlnPlot(tp_mergedata, 
            features = c("nFeature_RNA", "nCount_RNA"),
            ncol = 2)
}

# options(repr.plot.width=16, repr.plot.height=9)
f_show_QC(s09s)

f_sns_QC <- function(sn, sup_f, sup_c){
    # 以s09s为准
    for (lc_ba in names(s09s)){
        sn[[lc_ba]] <- subset(sn[[lc_ba]], subset = nFeature_RNA > 200 & nFeature_RNA < sup_f & nCount_RNA < sup_c)
    }
    sn
}

```

## 第六步 IntegrateData

```

f_FindVariableFeatures <- function(sn){
    # 以s09s为准
    for (lc_ba in names(s09s)){
        sn[[lc_ba]] <- NormalizeData(sn[[lc_ba]],
                          normalization.method = "LogNormalize", scale.factor = 10000)
        sn[[lc_ba]] <- FindVariableFeatures(sn[[lc_ba]],
                          selection.method = "vst", nfeatures = 2000)
    }
    sn
}

s09s <- f_FindVariableFeatures(s09s)
s11s <- f_FindVariableFeatures(s11s)
s12s <- f_FindVariableFeatures(s12s)
s13s <- f_FindVariableFeatures(s13s)

# 7、进行标准化并筛选可变基因
for (lc_i in 1:length(x10s)) {
    x10s[[lc_i]] <- NormalizeData(x10s[[lc_i]],
                      normalization.method = "LogNormalize", scale.factor = 10000)
    x10s[[lc_i]] <- FindVariableFeatures(x10s[[lc_i]],
                      selection.method = "vst", nfeatures = 2000)
}


f_IntegrateData <- function(olist){
    # 8、鉴定用于多样本数据整合的anchors
    tp_anchors <- FindIntegrationAnchors(object.list = olist, dims = 1:30, 
                          reduction = "cca")
    gc()
    # 9、整合多样本数据集 注意 内存需求大于16g！！！，不足请设置足够的swap空间再运行（总内存32g是够的）
    tp_int <- IntegrateData(anchorset = tp_anchors)
    gc()
    tp_int
}

scRNAs <- list()
scRNAs[['BA213']] <- f_IntegrateData(list(x10s$BA213,s09s$BA213,s11s$BA213,s12s$BA213,s13s$BA213))
scRNAs[['BA04']] <- f_IntegrateData(list(x10s$BA04,s09s$BA04,s11s$BA04,s12s$BA04,s13s$BA04))
scRNAs[['BA06']] <- f_IntegrateData(list(s09s$BA06,s11s$BA06,s12s$BA06,s13s$BA06))
scRNAs[['BA09']] <- f_IntegrateData(list(x10s$BA09,s09s$BA09,s11s$BA09,s12s$BA09,s13s$BA09))
scRNAs[['BA21']] <- f_IntegrateData(list(x10s$BA21,s09s$BA21,s11s$BA21,s12s$BA21,s13s$BA21))
scRNAs[['BA22']] <- f_IntegrateData(list(x10s$BA22,s09s$BA22,s11s$BA22,s12s$BA22,s13s$BA22))
scRNAs[['BA37']] <- f_IntegrateData(list(s09s$BA37,s11s$BA37,s12s$BA37,s13s$BA37))
scRNAs[['BA39']] <- f_IntegrateData(list(x10s$BA39,s09s$BA39,s11s$BA39,s12s$BA39,s13s$BA39))
scRNAs[['BA40']] <- f_IntegrateData(list(x10s$BA40,s09s$BA40,s11s$BA40,s12s$BA40,s13s$BA40))
scRNAs[['BA41']] <- f_IntegrateData(list(s09s$BA41,s11s$BA41,s12s$BA41,s13s$BA41))
scRNAs[['BA42']] <- f_IntegrateData(list(s09s$BA42,s11s$BA42,s12s$BA42,s13s$BA42))
scRNAs[['BA44']] <- f_IntegrateData(list(x10s$BA44,s09s$BA44,s11s$BA44,s12s$BA44,s13s$BA44))
scRNAs[['BA45']] <- f_IntegrateData(list(x10s$BA45,s09s$BA45,s11s$BA45,s12s$BA45,s13s$BA45))

scRNAs

```

## 第七步 保存数据

```
saveRDS(scRNAs, 'hBLA_scRNAs.rds')
```