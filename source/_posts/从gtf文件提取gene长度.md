---
title: 从GTF文件提取Gene长度
tags: []
id: '1917'
categories:
  - - uncategorized
date: 2022-07-18 10:58:48
---

先从TCGA的数据中提取一份[标准的基因长度](https://file-cdn.limour.top/bix/gene_length_tcga_22.07.18.csv.gz)，作为正确结果的标准，然后开始提取工作。

## Shell预处理

```bash
#!/bin/bash
GFT=/home/jovyan/upload/zl_liu/star/gencode.v36.primary_assembly.annotation.gtf
 
# 提取exon出来：
awk '{if ($3=="exon") print $0}' $GFT  > exon.gtf
# 去掉 exon.gtf 文件中的双引号：
sed -i 's/"//g' exon.gtf
# 提取需要的信息到txt：
awk 'BEGIN{FS="\t ;";OFS="\t"}{print $1,$3,$4,$5,$7,$10,$5-$4+1}' exon.gtf > exon.txt
```

*   [nano 12.sh](https://blog.csdn.net/weixin_51192038/article/details/122195717)
*   chmod +x 12.sh
*   ./12.sh && rm exon.gtf

## R进行组装

```R
library(tidyverse)
df <- read.table('exon.txt', header = F, sep = '\t', allowEscapes = T, quote = '')
df
df <- df %>% group_by(V6)
df <- df %>% summarise(length=sum(V7), chr=unique(V1), strand=unique(V5), start=min(V3), end=max(V4))
df[['width']] = df[['end']] - df[['start']]
df
write.csv(df,'gene_length_human.csv')
```

![](https://img.limour.top/archives_2023/2022/07/18/62d53bed5be33.webp)

和TCGA结果一致，说明结果可信，且length理论上比width更好