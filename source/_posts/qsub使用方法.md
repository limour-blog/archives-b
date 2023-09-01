---
title: 复旦高性能计算 (二) qsub使用方法
tags:
  - Linux
  - qsub
  - 高性能计算节点
id: '637'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-09-19 19:43:00
---

## 管理使用

*   查看所有任务：`qstat`
*   删除某个任务：`qdel <Job ID>`
*   交互使用：`qsub -I [-q <batch>]`
*   登入指定节点：`qsub -l nodes=cu05 -I`

## 提交任务

### 示例一 `jupyter.pbs`

*   创建 `/home/rqzhang/jupyter.pbs` 写入以下内容：

```
#!/bin/bash
#PBS -q batch
#PBS -V
#PBS -o /home/rqzhang/jupyter.out
#PBS -e /home/rqzhang/jupyter.err
#PBS -l nodes=1:ppn=8
#PBS -r y
cd /home/rqzhang
/home/rqzhang/zlliu/bin/frpc -c /home/rqzhang/zlliu/frp/frpc.ini &
jupyter notebook --ip='0.0.0.0' --port=18888 --no-browser --NotebookApp.token="<token>" --NotebookNotary.db_file=':memory:'
echo $HOME
```

*   提交任务： `qsub ./jupyter.pbs`

### 示例二 `SingleR.pbs`

*   创建 ``/home/rqzhang/`SingleR.pbs` `` 写入以下内容：

```
#!/bin/bash
#PBS -q fat
#PBS -V
#PBS -o /home/rqzhang/SingleR.out
#PBS -e /home/rqzhang/SingleR.err
#PBS -l nodes=1:ppn=32
#PBS -r y
cd /home/rqzhang
Rscript /home/rqzhang/r.pbs/21.07.15.10x.hM1.se.R
echo $HOME
```

*   创建 `/home/rqzhang/r.pbs/21.07.15.10x.hM1.se.R` 写入以下内容：

```
# 0、加载程辑包
library(Seurat)
library(dplyr)
library(patchwork)

# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"
# 配置结果保存路径
output_path = "~/zlliu/R_output/21.07.15.10x.hM1.se/"

# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
print(getwd())

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

# 6、QC
scRNA[["percent.mt"]] <- PercentageFeatureSet(scRNA, pattern = "^MT-")

# 6.2、分割数据以便QC
x10s <- SplitObject(scRNA, split.by = 'ident')
rm(scRNA)
gc()

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

# 加载 SingleR
library(SingleR)
library(SummarizedExperiment)
library(scater)
library(BiocParallel) # 并行计算加速

# 加载参考数据集 大概3分钟
if (!("hM1.se" %in% ls())){
    hM1.se <- readRDS("/home/rqzhang/zlliu/R_data/human_M1_10x/hM1.se.rds")
}

f_get_pred <- function(scRNA, lc_i){
    
    # 如果已经计算过了，就不再重复计算，直接返回上一次的结果
    if(file.exists(sprintf( "%02d_hM1_subclass.txt", lc_i))){
        result_main_hpca <- read.table(sprintf( "%02d_hM1_subclass.txt", lc_i), stringsAsFactors = F)
        return (result_main_hpca)
    }
    
    # 导出原始counts矩阵
    test.count=as.data.frame(scRNA[["RNA"]]@counts)
    
    # 保留共同基因
    common_hpca <- intersect(rownames(test.count), rownames(hM1.se))
    lc_hM1.se <- hM1.se[common_hpca,]
    test.count_forhpca <- test.count[common_hpca,]
    gc()
    
    # 生成test数据集
    test.count_forhpca.se <- SummarizedExperiment(assays=list(counts=test.count_forhpca))
    test.count_forhpca.se <- logNormCounts(test.count_forhpca.se)
    
    # 进行分类预测 大概一小时
    pred.main.hpca <- SingleR(test = test.count_forhpca.se, ref = lc_hM1.se, labels = hM1.se$meta$subclass_label, BPPARAM=MulticoreParam(32))
    # 32 CPUs
    gc()
    
    #构造返回结果
    result_main_hpca <- as.data.frame(pred.main.hpca$labels)
    result_main_hpca$CB <- rownames(pred.main.hpca)
    colnames(result_main_hpca) <- c('hM1_subclass', 'CB')
    write.table(result_main_hpca, sprintf( "%02d_hM1_subclass.txt", lc_i)) #保存下来，方便以后调用
    result_main_hpca
}

tp_sc <- x10s
# rm(x10s)
gc()
tp_sc_st <- 1
tp_sc_len <- length(tp_sc)
for (lc_i in tp_sc_st:tp_sc_len) {
    print(lc_i)
    f_get_pred(tp_sc[[lc_i]], lc_i)
}
```

*   提交任务： ```qsub ./`` `SingleR.pbs` `` ```

### 示例三 `cellphonedb.pbs`

*   创建 ``/home/rqzhang/`cellphonedb.pbs` `` 写入以下内容：

```
#!/bin/bash
#PBS -q fat
#PBS -V
#PBS -o /home/rqzhang/cellphonedb.out
#PBS -e /home/rqzhang/cellphonedb.err
#PBS -l nodes=1:ppn=32
#PBS -r y

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_01
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_02
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_03
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_04
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_05
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_06
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_07
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_08
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_output/21.07.15.10x.hM1.se/SingleR_09
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

echo $HOME
```

*   提交任务： `````qsub ./```` ``` `` `cellphonedb.pbs` `` ``` ```` `````