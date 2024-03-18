---
title: 二代测序数据处理（二）从CleanData到表达矩阵
tags: []
id: '1475'
categories:
  - - 生物信息学
  - - 组织测序分析
date: 2022-01-25 00:52:05
---

## 第一步 准备好数据

*   CleanData：由测序公司提供

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-17.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-17.webp)

内有 name\_R1.fq.gz 和 name\_R2.fq.gz

*   对应物种的索引文件：[二代测序数据处理（一）数据格式说明](https://limour.top/1391.html)

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-15.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-15.webp)

大鼠的索引文件

*   对应物种的染色体模板

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-20.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-20.webp)

上面的染色体模板命名不规范，使用下面脚本修改

```shell
#!/bin/bash
for file in `ls /home/lxb/gene_template/rat/gene/Rattus_norvegicus.Rnor_6.0.dna.chromosome.*.fa`
do
    newFile='/home/gene/xuchen/ratgene/'`echo $file  sed 's/\(.*\)\/.*\.\([0-9XYMT]\+.fa\)$/\2/'`
    echo $newFile
    cp $file $newFile
done
```

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-22.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-22.webp)

规范后的染色体命名

*   对应物种的GFT注释文件

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-21.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-21.webp)

[两份注释文件的差别](https://www.biostars.org/p/217700/)，一般使用不带chr的文件

## 第二步 生成排好序的bam文件

*   nohup /home/gene/xuchen/gp1.bash > /home/gene/xuchen/1.log 2>&1 &

```shell
#!/bin/sh
#设置CleanData存放目录
export CLEAN=/home/gene/upload/xc-lz/gp
#设置idx存放目录
export HISAT_IDX=/home/lxb/gene_template/rat/idx/rat.hisat2.idx
#设置这一步的输出目录
export WORK=/home/gene/xuchen/output_gp
  
export CDIR=$(basename `pwd`)
echo $CDIR
echo $CLEAN
for file in $CLEAN/*
do
echo $file
SAMPLE=${file##*/}
echo $SAMPLE
r1=${SAMPLE}"_R1.fq.gz"
r2=${SAMPLE}"_R2.fq.gz"
echo $r1
echo $r2
hisat2 -p 12 --dta-cufflinks -x $HISAT_IDX -1 $CLEAN/$SAMPLE/$r1 -2 $CLEAN/$SAMPLE/$r2 -S $WORK/$SAMPLE.sam
samtools view -bS $WORK/${SAMPLE}".sam" > $WORK/${SAMPLE}".bam"
# 内存使用量为 6000M * 6 = 36G
samtools sort -m 6000M -@ 6 -o $WORK/${SAMPLE}".sorted.bam" $WORK/${SAMPLE}".bam"
#没用的东西删掉
rm $WORK/${SAMPLE}".sam"
rm $WORK/${SAMPLE}".bam"
done
```

*   查看 1.log 确保匹配率高于80%，下图为 95.01%

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-18.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-18.webp)

*   最终结果

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-19.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-19.webp)

## 第三步 生成基因矩阵

*   nohup /home/gene/xuchen/gp2.bash > /home/gene/xuchen/2.log 2>&1 &

```shell
#上一步的输出目录
export CLEAN_OUT=/home/gene/xuchen/output_gp
cd $CLEAN_OUT
#规范命名后的染色体模板数据
export GRCh38=/home/gene/xuchen/ratgene
#GTF数据，一般使用不带chr的
export GTF=/home/lxb/gene_template/rat/gtf/Rattus_norvegicus.Rnor_6.0.90.gtf
#整理样本列表, 空格分隔
export SAMPLES="A34 C15 C41 C8 M7 X24 X28"
#用来生csv的表头，逗号分隔，与样本列表对应
export LABLES="A34,C15,C41,C8,M7,X24,X28"
#设置输出目录
export WORK=/home/gene/xuchen/diffout_gp

export LIST=""
for file in $SAMPLES
do
LIST=$LIST" "$CLEAN_OUT/$file".sorted.bam"
done
echo $LIST

cuffdiff -L $LABLES -o $WORK -b $GRCh38 -p 12 -u $GTF $LIST
```

*   查看处理进度 **tail -f** /home/gene/xuchen/2.log
*   更简单的代码：

```shell
#!/bin/bash
#上一步的输出目录
export CLEAN_OUT=/home/gene/xuchen/output_gp
#设置这一步的输出目录
export WORK=/home/gene/xuchen/diffout_gp
#规范命名后的染色体模板数据
export GRCh38=/home/gene/xuchen/ratgene
#GTF数据，一般使用不带chr的
export GTF=/home/lxb/gene_template/rat/gtf/Rattus_norvegicus.Rnor_6.0.90.gtf

LIST=""
LABLES=""
for file in `ls $CLEAN_OUT`
do 
file=`echo $file  sed 's/\(.*\).sorted.bam$/\1/'`
echo $file
LIST=$LIST" "$CLEAN_OUT/$file".sorted.bam"
LABLES=$LABLES","$file
done
LABLES=`echo ${LABLES: 1}`
echo $LIST
echo $LABLES

cd $CLEAN_OUT
cuffdiff -L $LABLES -o $WORK -b $GRCh38 -p 12 -u $GTF $LIST
```

*   最终结果

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-23.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-23.webp)

## 第四步 转换成symbol的csv

```r
gp <- read.table('~/gene/xuchen/diffout_gp/genes.fpkm_tracking', header = T)
sampleN <- stringr::str_detect(colnames(gp),'FPKM')
sampleN <- colnames(gp)[sampleN]
gp_filter <- gp[,c('gene_id', 'gene_short_name', sampleN)]
write.csv(gp_filter, file='gp.csv')
```

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-24.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-24.webp)

gp的内容

[![](https://img.limour.top/archives_2023/blog_wp/2022/01/image-25.webp)](https://img.limour.top/archives_2023/blog_wp/2022/01/image-25.webp)

gp.csv的内容