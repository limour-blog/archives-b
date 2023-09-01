---
title: cellphoneDB (一) 使用流程
tags: []
id: '850'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-10-02 23:20:31
---

## 第一步 导出数据

```
f_prepare4cellphoneDB <- function(lc_scRNA, lc_dir, lc_className){
    if (!file.exists(lc_dir)){dir.create(lc_dir)}
    # 生成 count.txt 
    write.table(as.matrix(lc_scRNA@assays$RNA@data), file.path(lc_dir,'cellphonedb_count.txt'), sep='\t', quote=F)
    # 生成 meta.txt
    lc_meta_data <- cbind(rownames(lc_scRNA@meta.data), lc_scRNA@meta.data[, lc_className, drop=F])
    lc_meta_data <- as.matrix(lc_meta_data)
    lc_meta_data[is.na(lc_meta_data)] = "Unkown" #  细胞类型中不能有NA
    write.table(lc_meta_data, file.path(lc_dir,'cellphonedb_meta.txt'), sep='\t', quote=F, row.names=F)
}
f_prepare4cellphoneDB(sc_Neuron,"Neuron", "hmca_class")
f_prepare4cellphoneDB(sc_Neuron,"Neuron_br", "orig.ident")
```

## 第二步 提交运算

```
#!/bin/bash
#PBS -q batch
#PBS -V
#PBS -o /home/rqzhang/cellphonedb.out
#PBS -e /home/rqzhang/cellphonedb.err
#PBS -l nodes=1:ppn=32
#PBS -r y
 
cd /home/rqzhang/zlliu/R_data/21.10.01.10x/Neuron
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_data/21.10.01.10x/Neuron_br
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

```

## 第三步 可视化

```
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

# Rearrange data column sequence
library(dplyr)
f_cDB_order_sequence <- function(lc_df){
    da <- data.frame()
    df <- subset(lc_df, receptor_a == 'True' & receptor_b == 'False'  receptor_a == 'False' & receptor_b == 'True')
    for(i in 1:length(df$gene_a)){
    sub_data <- df[i, ]
    if(sub_data$receptor_b=='False'){
        if(sub_data$receptor_a=='True'){
            old_names <- colnames(sub_data)
            my_list <- strsplit(old_names[-c(1:11)], split="\\")
            my_character <- paste(sapply(my_list, '[[', 2L), sapply(my_list, '[[', 1L), sep='')
            new_names <- c(names(sub_data)[1:4], 'gene_b', 'gene_a', 'secreted', 'receptor_b', 'receptor_a', "annotation_strategy", "is_integrin", my_character)
            sub_data = dplyr::select(sub_data, new_names)
            # print('Change sequence!!!')
            names(sub_data) <- old_names
            da = rbind(da, sub_data) 
        }
    }else{
            da = rbind(da, sub_data)
    }
    }
    return(da)
}

f_cDB_mergePandM <- function(means_order, pvals_order){
    means_sub <- means_order[, c('interacting_pair', colnames(means_order)[-c(1:11)])]
    pvals_sub <- pvals_order[, c('interacting_pair', colnames(means_order)[-c(1:11)])]
    means_gather <- tidyr::gather(means_sub, celltype, mean_expression, names(means_sub)[-1])
    pvals_gather <- tidyr::gather(pvals_sub, celltype, pval, names(pvals_sub)[-1])
    mean_pval <- dplyr::left_join(means_gather, pvals_gather, by = c('interacting_pair', 'celltype'))
    mean_pval
}

f_readcellphoneDB <- function(lc_dir){
    res = list()
    res$pvals <- f_cDB_order_sequence(read.delim(file.path(lc_dir, "out","pvalues.txt"), check.names = FALSE))
    res$means <- f_cDB_order_sequence(read.delim(file.path(lc_dir, "out", "means.txt"), check.names = FALSE))
    res$s_means <- read.delim(file.path(lc_dir, "out", "significant_means.txt"), check.names = FALSE)
    res$m_p <- f_cDB_mergePandM(res$means, res$pvals)
    lc_tp <- res$m_p %>% dplyr::select(interacting_pair, celltype, pval) %>% tidyr::spread(key=celltype, value=pval)
    lc_sig_pairs <- lc_tp[which(rowSums(lc_tp<=0.05)!=0), ]
    res$s_m_p <- subset(res$m_p, interacting_pair %in% lc_sig_pairs$interacting_pair)
    res
}

f_cDB_dotplot <- function(lc_m_p){
    lc_m_p %>% ggplot(aes(x=interacting_pair, y=celltype)) +
    # geom_point(aes(color=log2(mean_expression), size=pval)) +
    # scale_size(trans = 'reverse') +
    geom_point(aes(color=log2(mean_expression), size=-log10(pval+1*10^-3)) ) +
    guides(colour = guide_colourbar(order = 1),size = guide_legend(order = 2)) +
    labs(x='', y='') +
    scale_color_gradientn(name='Expression level \n(log2 mean expression \nmolecule1, molecule2)', colours = terrain.colors(100)) +
    # scale_color_gradient2('Expression level \n(log2 mean expression \nmolecule1, molecule2)', low = 'blue', mid = 'yellow', high = 'red') +
    theme(axis.text.x= element_text(angle=45, hjust=1)) +
    # coord_flip() +
    theme(
    panel.border = element_rect(color = 'black', fill = NA),
    panel.grid.major.x = element_blank(),
    panel.grid.major.y = element_blank(),
    panel.grid.minor.x = element_blank(),
    panel.grid.minor.y = element_blank(),
    panel.background   = element_blank(),
    axis.title.x       = element_blank(),
    axis.title.y       = element_blank(),
    axis.ticks         = element_blank()
    # plot.title         = element_text(hjust = 0.5),
    # legend.position = 'bottom' # guides(fill = guide_legend(label.position = "bottom"))
    # legend.position    = "bottom"
    # axis.text.y.right  = element_text(angle=270, hjust=0.5)
    ) +
    theme(legend.key.size = unit(0.4, 'cm'), #change legend key size
    # legend.key.height = unit(1, 'cm'), #change legend key height
    # legend.key.width = unit(1, 'cm'), #change legend key width
    legend.title = element_text(size=9), #change legend title font size
    legend.text = element_text(size=8)) #change legend text font size
}

br_d <- f_readcellphoneDB('Neuron_br')
tp_img <- f_cDB_dotplot(subset(br_d $s_m_p, pval<0.05))
f_image_output('Neuron_br',tp_img, width = 1080,height = 1080)

```