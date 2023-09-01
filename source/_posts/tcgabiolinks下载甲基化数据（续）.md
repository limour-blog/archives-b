---
title: TCGAbiolinks下载甲基化数据（续）
tags: []
id: '2018'
categories:
  - - uncategorized
date: 2022-09-10 13:09:17
---

之前[TCGAbiolinks下载甲基化数据](https://occdn.limour.top/2310.html)的hg19的数据下载好了，现在继续进行分析

## 安装包

*   conda create -n rGADEM -c conda-forge r-base=3.6 -y
*   conda activate rGADEM
*   conda install -c bioconda bioconductor-motiv -y
*   conda install -c conda-forge libopenblas -y
*   conda install -c conda-forge openblas -y
*   install.packages('Matrix')
*   conda install -c conda-forge r-irkernel -y
*   Rscript -e "IRkernel::installspec(name='rGADEM', displayname='r-rGADEM')"

## 定义函数

```R
require(rGADEM)
require(GenomicRanges)
require(motifStack)
require(dplyr)
f_gadem <- function(probes, seqname, delta_start = 100, delta_end = 100){
    probes_tmp <- subset(probes, seqnames==seqname)
    sequence <- GRanges(
      seqnames = rep(seqname,length(probes_tmp)),
      IRanges(
        start = ranges(probes_tmp) %>% as.data.frame() %>% dplyr::pull("start") - 100,
        end = ranges(probes_tmp) %>% as.data.frame() %>% dplyr::pull("end") + 100), 
      strand = "*"
    )
    gadem <- GADEM(sequence, verbose = FALSE, genome = BSgenome.Hsapiens.UCSC.hg19::Hsapiens)
    gadem
}
```

```R
f_gadem_one <- function(probes, seqname, delta_start = 100, delta_end = 100){
    probes_tmp <- subset(probes, seqnames==seqname)
    res <- list(
        seqnames = rep(seqname,length(probes_tmp)),
        start = ranges(probes_tmp) %>% as.data.frame() %>% dplyr::pull("start") - 100,
        end = ranges(probes_tmp) %>% as.data.frame() %>% dplyr::pull("end") + 100
    )
    res
}
f_gadem <- function(probes, all_seqN=NULL, delta_start = 100, delta_end = 100){
    if(is.null(all_seqN)){
        all_seqN <- as.character(unique(seqnames(probes)))
    }
    res_seqnames <- NULL
    res_start <- NULL
    res_end <- NULL
    for(x in all_seqN){
        res <- f_gadem_one(probes, x, delta_start, delta_end)
        res_seqnames <- c(res_seqnames, res$seqnames)
        res_start <- c(res_start, res$start)
        res_end <- c(res_end, res$end)
    }
    sequence <- GRanges(
      seqnames = res_seqnames,
      IRanges(
        start = res_start,
        end = res_end), 
      strand = "*"
    )
    gadem <- GADEM(sequence, verbose = FALSE, genome = BSgenome.Hsapiens.UCSC.hg19::Hsapiens)
    gadem
}
```

## 发现模体

```R
saveRDS(rowRanges(res_data), 'probes_hg19.rds')
 
probes <- readRDS('probes_hg19.rds')
gadem <- f_gadem(probes, 'chr1')
nMotifs(gadem) # 找到的模体数量
nOccurrences(gadem) # 出现的次数
consensus(gadem) # 查看模体的一致性序列
# 展示模体 logo 图
pwm <- getPWM(gadem)
pfm  <- new("pfm",mat=pwm[[1]],name="Novel Site 1")
plotMotifLogo(pfm)
saveRDS(pwm, 'chr1_pwm.rds')
```

## 配对分析

```R
library(MotIV)
pwm <- readRDS('chr1_pwm.rds')
analysis.jaspar <- motifMatch(pwm)
summary(analysis.jaspar)
options(repr.plot.width=18, repr.plot.height=12)
plot(analysis.jaspar, ncol=2, top=5, rev=FALSE, main="", bysim=TRUE, cex=0.4)
```