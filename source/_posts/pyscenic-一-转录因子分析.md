---
title: pyscenic (一) 转录因子分析
tags: []
id: '1063'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-10-15 01:50:00
---

## 第一步 安装

```
pip3.8 install wheel -i https://pypi.tuna.tsinghua.edu.cn/simple
pip3.8 install pyscenic -i https://pypi.tuna.tsinghua.edu.cn/simple
pip3.8 install seaborn -i https://pypi.tuna.tsinghua.edu.cn/simple
pip3.8 install scanpy -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 第二步 跑示例教程

```
import os
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

```

```
# 配置数据路径
root_path = os.path.expanduser("~/zlliu/R_output/21.10.22.pyscenic")
 
# 配置结果保存路径
output_path = root_path
if not os.path.isdir(output_path):
    os.makedirs(output_path)

```

```
DATA_FOLDER="./tmp"
RESOURCES_FOLDER="./resources"
DATABASE_FOLDER = "."
DATABASES_GLOB = os.path.join(DATABASE_FOLDER, "mm9-*.mc9nr.feather")
MOTIF_ANNOTATIONS_FNAME = os.path.join(RESOURCES_FOLDER, "motifs-v9-nr.mgi-m0.001-o0.0.tbl")
MM_TFS_FNAME = os.path.join(RESOURCES_FOLDER, 'mm_tfs.txt')
SC_EXP_FNAME = os.path.join(RESOURCES_FOLDER, "GSE60361_C1-3005-Expression.txt")
REGULONS_FNAME = os.path.join(DATA_FOLDER, "regulons.p")
MOTIFS_FNAME = os.path.join(DATA_FOLDER, "motifs.csv")
```

```
!wget https://ftp.ncbi.nlm.nih.gov/geo/series/GSE60nnn/GSE60361/suppl/GSE60361_C1-3005-Expression.txt.gz
if not os.path.isdir(RESOURCES_FOLDER):
    os.makedirs(RESOURCES_FOLDER)
!gunzip -c ./GSE60361_C1-3005-Expression.txt.gz > $RESOURCES_FOLDER/GSE60361_C1-3005-Expression.txt
```

```
ex_matrix = pd.read_csv(SC_EXP_FNAME, sep='\t', header=0, index_col=0).T
ex_matrix.shape
```

```
!wget https://resources.aertslab.org/cistarget/motif2tf/motifs-v9-nr.hgnc-m0.001-o0.0.tbl
!wget https://resources.aertslab.org/cistarget/motif2tf/motifs-v9-nr.mgi-m0.001-o0.0.tbl
!wget https://raw.githubusercontent.com/aertslab/pySCENIC/master/resources/lambert2018.txt
BASEFOLDER_NAME = '.'

MOTIFS_HGNC_FNAME = os.path.join(BASEFOLDER_NAME, 'motifs-v9-nr.hgnc-m0.001-o0.0.tbl')
MOTIFS_MGI_FNAME = os.path.join(BASEFOLDER_NAME, 'motifs-v9-nr.mgi-m0.001-o0.0.tbl')
CURATED_TFS_HGNC_FNAME = os.path.join(BASEFOLDER_NAME, 'lambert2018.txt')

OUT_TFS_HGNC_FNAME = os.path.join(RESOURCES_FOLDER, 'hs_hgnc_tfs.txt')
OUT_TFS_HGNC_FNAME = os.path.join(RESOURCES_FOLDER, 'hs_hgnc_curated_tfs.txt')
OUT_TFS_MGI_FNAME = os.path.join(RESOURCES_FOLDER, 'mm_mgi_tfs.txt')

df_motifs_mgi = pd.read_csv(MOTIFS_MGI_FNAME, sep='\t')
mm_tfs = df_motifs_mgi.gene_name.unique()
with open(OUT_TFS_MGI_FNAME, 'wt') as f:
    f.write('\n'.join(mm_tfs) + '\n')
len(mm_tfs)

df_motifs_hgnc = pd.read_csv(MOTIFS_HGNC_FNAME, sep='\t')
hs_tfs = df_motifs_hgnc.gene_name.unique()
with open(OUT_TFS_HGNC_FNAME, 'wt') as f:
    f.write('\n'.join(hs_tfs) + '\n')
len(hs_tfs)

with open(CURATED_TFS_HGNC_FNAME, 'rt') as f:
    hs_curated_tfs = list(map(lambda s: s.strip(), f.readlines()))
len(hs_curated_tfs)

hs_curated_tfs_with_motif = list(set(hs_tfs).intersection(hs_curated_tfs))
len(hs_curated_tfs_with_motif)

with open(OUT_TFS_HGNC_FNAME, 'wt') as f:
    f.write('\n'.join(hs_curated_tfs_with_motif) + '\n')

```

```
tf_names = load_tf_names(MM_TFS_FNAME)
```

```
!wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm9/refseq_r45/mc9nr/gene_based/mm9-tss-centered-5kb-10species.mc9nr.feather
!wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm9/refseq_r45/mc9nr/gene_based/mm9-500bp-upstream-7species.mc9nr.feather
!wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm9/refseq_r45/mc9nr/gene_based/mm9-500bp-upstream-10species.mc9nr.feather
!wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm9/refseq_r45/mc9nr/gene_based/mm9-tss-centered-10kb-7species.mc9nr.feather
!wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm9/refseq_r45/mc9nr/gene_based/mm9-tss-centered-10kb-10species.mc9nr.feather
!wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm9/refseq_r45/mc9nr/gene_based/mm9-tss-centered-5kb-7species.mc9nr.feather
```

```
db_fnames = glob.glob(DATABASES_GLOB)
def name(fname):
    return os.path.splitext(os.path.basename(fname))[0]
dbs = [RankingDatabase(fname=fname, name=name(fname)) for fname in db_fnames]
dbs
```

```
adjacencies = grnboost2(ex_matrix, tf_names=tf_names, verbose=True)
with open('adjacencies.pydata','wb') as f:
    pickle.dump(adjacencies,f)
modules = list(modules_from_adjacencies(adjacencies, ex_matrix))
with open('modules.pydata','wb') as f:
    pickle.dump(modules,f)
with ProgressBar():
    df = prune2df(dbs, modules, MOTIF_ANNOTATIONS_FNAME)
with open('df.pydata','wb') as f:
    pickle.dump(df,f)
from gc import collect as gc
gc()
regulons = df2regulons(df)
with open('regulons.pydata','wb') as f:
    pickle.dump(regulons,f)
auc_mtx = aucell(ex_matrix, regulons, num_workers=4)
sns.clustermap(auc_mtx, figsize=(8,8))
```