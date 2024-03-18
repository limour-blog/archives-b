---
title: Seurat (八) BoxPlot：不同组间不同细胞的基因表达
tags: []
id: '1231'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-11-03 19:20:15
---

```r
f_image_output <- function(fileName, image, width=1920, height=1080, lc_pdf=T, lc_resolution=72){
    if(lc_pdf){
        width = width / lc_resolution
        height = height / lc_resolution
        pdf(paste(fileName, ".pdf", sep=""), width = width, height = height)
    }else{
        png(paste(fileName, ".png", sep=""), width = width, height = height)
    }
    print(image)
    dev.off()
}

options(repr.plot.width=12, repr.plot.height=12)
options(ggrepel.max.overlaps = Inf)
f_icg_boxp <- function(lc_exp, lc_icg, lc_g, title = NULL){
    lc_exp_L = melt(lc_exp[lc_icg, rownames(lc_g)])
    lc_exp_L <- cbind(lc_exp_L, rownames(lc_exp_L))
    colnames(lc_exp_L)=c('value','sample')
    if(is.data.frame(lc_g)){
        lc_exp_L$group = lc_g[[1]]
    }else{
        lc_exp_L$group = lc_g
    }
    p=ggplot(lc_exp_L,aes(x=group,y=value,fill=group))+geom_boxplot()
    p=p+stat_summary(fun="mean",geom="point",shape=23,size=3,fill="red")
    p=p+theme_set(theme_set(theme_bw(base_size=20)))
    p=p+theme(text=element_text(face='bold'),axis.text.x=element_text(angle=30,hjust=1),axis.title=element_blank())
    if (length(title) == 0){title = lc_icg}
    p=p+ggtitle(title)+theme(plot.title = element_text(hjust = 0.5))
    p
}

f_bp_gcg <- function(sObject, lc_groupN, lc_labelN, gG){
    ss <- SplitObject(sObject, split.by = lc_groupN)
    lc_N =  names(ss)[1]
    p = f_icg_boxp(ss[[lc_N]][['RNA']]@data, gG, ss[[lc_N]][[lc_labelN]], sprintf('%s in %s', gG, lc_N))
    for (lc_N in names(ss)[-1]){
        p = p + f_icg_boxp(ss[[lc_N]][['RNA']]@data, gG, ss[[lc_N]][[lc_labelN]], sprintf('%s in %s', gG, lc_N))
    }
    p + plot_layout(ncol = 3)
}

f_br_cluster_f <- function(sObject, lc_groupN){
    lc_filter <- unlist(unique(sObject[[lc_groupN]]))
    lc_filter <- lc_filter[!is.na(lc_filter)]
    lc_filter
}

f_metadata_removeNA <- function(sObject, lc_groupN){
    sObject@meta.data <- sObject@meta.data[colnames(sObject),]
    sObject <- subset(x = sObject, !!sym(lc_groupN)%in%f_br_cluster_f(sObject, lc_groupN))
    sObject
}

scRNA_split = readRDS("~/zlliu/R_output/21.09.21.SingleR/scRNA.rds")
scRNA_split <- f_metadata_removeNA(scRNA_split, 'Region')

f_bp_gcg(scRNA_split, 'Region', 'hM1_hmca_class', 'NRXN3')
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/下载-2.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/下载-2.webp)