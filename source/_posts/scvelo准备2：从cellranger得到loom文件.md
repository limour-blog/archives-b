---
title: scVelo准备2：从cellranger得到loom文件
tags: []
id: '2033'
categories:
  - - uncategorized
date: 2022-09-29 12:37:19
---

```shell
conda activate velocyto

#!/bin/bash
db=/opt/cellranger/refdata-gex-GRCh38-2020-A
work=/home/jovyan/upload/zl_liu/data/data/res
rmsk_gtf=/opt/cellranger/GRCh38_rmsk.gtf # 从genome.ucsc.edu下载 
cellranger_gtf=${db}/genes/genes.gtf
ls -lh $rmsk_gtf  $work $cellranger_gtf
for sample in ${work}/*;
do echo $sample
velocyto run10x -m $rmsk_gtf $sample $cellranger_gtf
done
```