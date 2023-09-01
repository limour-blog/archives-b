---
title: pyscenic (二) 转录因子分析
tags: []
id: '1111'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-10-23 13:42:47
---

## 前置要求

[pyscenic (一) 转录因子分析](https://limour.top/1063.html)

[pySCENIC](https://pyscenic.readthedocs.io/en/latest/index.html)

#### [aertslab](https://github.com/aertslab)/**[pySCENIC](https://github.com/aertslab/pySCENIC)**

## 第一步 准备数据库

*   设置工作目录

```
import os
# 配置数据路径
root_path = os.path.expanduser("~/zlliu/R_output/21.10.23.pyscenic")
 
# 配置结果保存路径
output_path = root_path
if not os.path.isdir(output_path):
    os.makedirs(output_path)

# 设置工作目录，输出文件将保存在此目录下
os.chdir(output_path) 
os.getcwd()
```

*   下载智人的转录因子列表 [https://github.com/aertslab/pySCENIC/tree/master/resources](https://github.com/aertslab/pySCENIC/tree/master/resources)

```
!wget https://raw.githubusercontent.com/aertslab/pySCENIC/master/resources/hs_hgnc_tfs.txt
```

*   下载 annotations file [https://pyscenic.readthedocs.io/en/latest/installation.html#auxiliary-datasets](https://pyscenic.readthedocs.io/en/latest/installation.html#auxiliary-datasets)

```
!wget https://resources.aertslab.org/cistarget/motif2tf/motifs-v9-nr.hgnc-m0.001-o0.0.tbl
```

*   下载 Auxiliary datasets [https://resources.aertslab.org/cistarget/](https://resources.aertslab.org/cistarget/)

```
!wget https://resources.aertslab.org/cistarget/databases/homo_sapiens/hg38/refseq_r80/mc9nr/gene_based/hg38__refseq-r80__10kb_up_and_down_tss.mc9nr.feather
```

## 第二步 输出 ex\_matrix

```
%%R
require(Seurat)
SeuratObject = subset(SeuratObject, CellType == 'fibroblast')
write.csv(as.matrix(SeuratObject[['RNA']]@counts), file = "ex_matrix.csv")
cellInfo = SeuratObject@metadata
```

## 第三步 计算

*   导入包和数据

```
import glob
import pickle
import pandas as pd
import numpy as np
 
from dask.diagnostics import ProgressBar
 
from arboreto.utils import load_tf_names
from arboreto.algo import grnboost2
 
from ctxcore.rnkdb import FeatherRankingDatabase as RankingDatabase
from pyscenic.utils import modules_from_adjacencies, load_motifs
from pyscenic.prune import prune2df, df2regulons
from pyscenic.aucell import aucell
 
import seaborn as sns
```

```
SC_EXP_FNAME = 'ex_matrix.csv'
TFS_FNAME = 'hs_hgnc_tfs.txt'
DATABASES_GLOB = "hg38*.mc9nr.feather"
MOTIF_ANNOTATIONS_FNAME = 'motifs-v9-nr.hgnc-m0.001-o0.0.tbl'

ex_matrix = pd.read_csv(SC_EXP_FNAME, sep=',', header=0, index_col=0).T
ex_matrix.shape

tf_names = load_tf_names(TFS_FNAME)

db_fnames = glob.glob(DATABASES_GLOB)
def name(fname):
    return os.path.splitext(os.path.basename(fname))[0]
dbs = [RankingDatabase(fname=fname, name=name(fname)) for fname in db_fnames]
dbs
```

*   计算 grn

```
adjacencies = grnboost2(ex_matrix, tf_names=tf_names, verbose=True)
```

```
with open('adjacencies.pydata','wb') as f:
    pickle.dump(adjacencies,f)
```

*   计算 ctx

```
modules = list(modules_from_adjacencies(adjacencies, ex_matrix))
with ProgressBar():
    df = prune2df(dbs, modules, MOTIF_ANNOTATIONS_FNAME)
regulons = df2regulons(df)
```

```
with open('modules.pydata','wb') as f:
    pickle.dump(modules,f)
with open('df.pydata','wb') as f:
    pickle.dump(df,f)
with open('regulons.pydata','wb') as f:
    pickle.dump(regulons,f)
```

*   计算 AUCell

```
regulons = [r.rename(r.name.replace('(+)',' ('+str(len(r))+'g)')) for r in regulons] # 第四步 导入方法二
auc_mtx = aucell(ex_matrix, regulons, num_workers=4)
```

```
with open('auc_mtx.pydata','wb') as f:
    pickle.dump(auc_mtx,f)
```

## 第四步 导入R中出图

[pySCENIC](https://www.jianshu.com/p/eccfe2d1b2c7)

*   导入方法一 （不推荐）

```
from pyscenic.export import export2loom
#  regulons = [r.rename(r.name.replace('(+)',' ('+str(len(r))+'g)')) for r in regulons]
export2loom(ex_mtx = ex_matrix, auc_mtx = auc_mtx, regulons=regulons, out_fname = "results.loom")
```

```
%%R
require(ggplot2)
require(SCENIC)
require(SCopeLoomR)

loom <- open_loom("results.loom", mode="r")

# Read information from loom file:
regulons_incidMat <- get_regulons(loom)
regulons <- regulonsToGeneLists(regulons_incidMat)
regulonsAUC <- get_regulonsAuc(loom)
regulonsAucThresholds <- get_regulonThresholds(loom)
embeddings <- get_embeddings(loom)

exprMat <- get_dgem(loom)
cellInfo <- get_cellAnnotation(loom)
clusterings <- get_clusterings_withName(loom)

close_loom(loom)
```

*   导入方法二 （推荐）

```
auc_mtx.to_csv('auc_mtx.tsv')
```

```
%%R
require(ggplot2)
require(SCENIC)
require(SCopeLoomR)
require(AUCell)

regulonAUC <- importAUCfromText('auc_mtx.tsv')
regulonAUC <- regulonAUC[onlyNonDuplicatedExtended(rownames(regulonAUC)),]
# cellInfo = SeuratObject@metadata
regulonActivity_byCellType <- sapply(split(rownames(cellInfo), cellInfo$CellType),
                                     function(cells) rowMeans(getAUC(regulonAUC)[,cells]))
regulonActivity_byCellType_Scaled <- t(scale(t(regulonActivity_byCellType), center = T, scale=T))
pheatmap::pheatmap(regulonActivity_byCellType_Scaled, #fontsize_row=3, 
                   color=colorRampPalette(c("blue","white","red"))(100), breaks=seq(-3, 3, length.out = 100),
                   treeheight_row=10, treeheight_col=10, border_color=NA)
```

[SCENIC—Single Cell rEgulatory Network Inference and Clustering](https://www.jianshu.com/p/b0fd795ad05c)