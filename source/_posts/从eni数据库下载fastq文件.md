---
title: 从ENI数据库下载fastq文件
tags: []
id: '2030'
categories:
  - - uncategorized
date: 2022-09-25 14:43:52
---

进入[ENA Browser](https://www.ebi.ac.uk/ena/browser/view/PRJNA573608?show=reads)，搜索对应的GSE号，进入study项目，选择TSV格式的**Download report**。

![](https://img-cdn.limour.top/2022/09/25/633061b015030.png)

从TSV表格中提取下载链接，一行一个写入url.txt，前面加上`ftp://`，接着使用`wget -c -i url.txt`下载

来自[科研小徐](https://www.jianshu.com/u/10a7837324db)的[文章](https://www.jianshu.com/p/98fc6c80c216)中的批量重命名脚本：

```bash
ls *.fastq.gzcut -d '_' -f 1while read i ;do (echo ${i}_1*.gz' will be moved to '${i}_S1_L001_R1_001.fastq.gz);done
ls *.fastq.gzcut -d '_' -f 1while read i ;do (echo ${i}_2*.gz' will be moved to '${i}_S1_L001_R2_001.fastq.gz);done
 
ls *.fastq.gzcut -d '_' -f 1while read i ;do (mv ${i}_1*.gz ${i}_S1_L001_R1_001.fastq.gz;mv ${i}_2*.gz ${i}_S1_L001_R2_001.fastq.gz);done
```