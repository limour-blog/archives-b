---
title: Seurat (五) 简单总结
tags: []
id: '886'
categories:
  - - Seurat教程
  - - 生物信息学
date: 2021-10-04 20:43:00
---

```
library(Matrix)
library(Seurat)
library(plyr)
library(dplyr)
library(patchwork)
library(purrr)

library(RColorBrewer)
library(ggplot2)
library(ggrepel)
blank_theme <- theme_minimal()+
    theme(
        axis.title.x = element_blank(),
        axis.text.x=element_blank(),
        axis.title.y = element_blank(),
        axis.text.y=element_blank(),
        panel.border = element_blank(),
        panel.grid=element_blank(),
        axis.ticks = element_blank(),
        plot.title=element_text(size=14, face="bold",hjust = 0.5)
    )
col_Paired <- colorRampPalette(brewer.pal(12, "Paired"))
f_pie <- function(lc_x, lc_main, lc_x_p = 1.3, lc_r = T){
    lc_cols <- col_Paired(length(lc_x))
    lc_v <- as.vector(100*lc_x)
    lc_df <- data.frame(type = names(lc_x), nums = lc_v)
    lc_df <- lc_df[order(lc_df$type),]
    lc_percent = sprintf('%0.2f%%',lc_df$nums)
    if(lc_r){
        lc_df$pos <- with(lc_df, 100-cumsum(nums)+nums/2)
    }else{
        lc_df$pos <- with(lc_df, cumsum(nums)-nums/2)
    } 
    lc_pie <- ggplot(data = lc_df, mapping = aes(x = 1, y = nums, fill = type)) + geom_bar(stat = 'identity')
#     print(lc_df)
#     print(lc_pie)
    lc_pie <- lc_pie + coord_polar("y", start=0, direction = 1) + scale_fill_manual(values=lc_cols) + blank_theme 
    lc_pie <- lc_pie + geom_text_repel(aes(x = lc_x_p, y=pos),label= lc_percent, force = T, 
                        arrow = arrow(length=unit(0.01, "npc")), segment.color = "#cccccc", segment.size = 0.5)
    lc_pie <- lc_pie + labs(title = lc_main)
    lc_pie
}

f_pie_metaN <- function(sObject, lc_group.by){
    tp_data <- prop.table(table(sObject[[lc_group.by]]))
    f_pie(tp_data, sprintf('Proportion of %s', lc_group.by))
}

f_UMAP_more <- function(sObject, lc_group.by, lc_reduction="umap"){
    res <- (DimPlot(sObject, reduction = lc_reduction, group.by = lc_group.by[1], label = T, repel = T, label.size = 6) + 
    labs(title = lc_group.by[1]))
    for(lc_i in 2:length(lc_group.by)){
        res <- res/
        (DimPlot(sObject, reduction = lc_reduction, group.by = lc_group.by[lc_i], label = T, repel = T, label.size = 6) + 
        labs(title = lc_group.by[lc_i]))
    }
    res
}

f_br_cluster_f <- function(sObject, lc_groupN){
    lc_filter <- unlist(unique(sObject[[lc_groupN]]))
    lc_filter <- lc_filter[!is.na(lc_filter)]
    lc_filter
}

f_br_cluster <- function(sObject, lc_groupN, lc_labelN, lc_prop = F){
    lc_all <- unique(sObject[[lc_labelN]])
    rownames(lc_all) <- lc_all[[1]]
    colnames(lc_all) <- "CB"
    lc_tp  <- SplitObject(subset(x = sObject, !!sym(lc_groupN)%in%f_br_cluster_f(sObject, lc_groupN)), split.by = lc_groupN)
    for(lc_i in 1:length(lc_tp)){
        if(lc_prop){
            lc_tp[[lc_i]] <- prop.table(table(lc_tp[[lc_i]][[lc_labelN]]))
        }else{
            lc_tp[[lc_i]] <- table(lc_tp[[lc_i]][[lc_labelN]])
        }
    }
    for(lc_name in names(lc_tp)){
        lc_all[[lc_name]] = 0
        lc_all[names(lc_tp[[lc_name]]), lc_name] = lc_tp[[lc_name]]
    }
    lc_all[,-1]
}
 
f_q2l <- function(lc_q){
    res  <- NULL
    for(lc_c in colnames(lc_q)){
        for(lc_r in rownames(lc_q)){
            res  <- rbind(res, c(group=lc_c, label=lc_r, value=lc_q[lc_r,lc_c]))
        }
    }
    res <- data.frame(res)
    res$value = as.numeric(res$value)
    res
}
 
library(RColorBrewer)
library(ggplot2)
col_Paired <- colorRampPalette(brewer.pal(12, "Paired"))
f_q_frequnency <- function(lc_q){
    ggplot(f_q2l(lc_q),mapping = aes(group,value,fill=label))+
    geom_bar(stat='identity',position='fill') + scale_fill_manual(values= col_Paired(nrow(lc_q)))+
    labs(x = 'group',y = 'frequnency') +
    theme(axis.title =element_text(size = 16),axis.text =element_text(size = 14, color = 'black'))+
    theme(axis.text.x = element_text(angle = 45, hjust = 1))+coord_flip() 
}

f_DEG_Volcano <- function(lc_logFC, lc_p, lc_gene, Threshold_logFC = 1, Threshold_p = 0.05, lc_rep=1:10){
    col_vector = rep(rgb(108, 200, 228, maxColorValue = 255), length(lc_logFC))
    col_vector[lc_p < Threshold_p & lc_logFC > Threshold_logFC] = rgb(226, 61, 75, maxColorValue = 255)
    col_vector[lc_p < Threshold_p & lc_logFC < -Threshold_logFC] = rgb(232, 168, 71, maxColorValue = 255)
    lc_p[lc_p < 1e-10] = 1e-10
    lc_p[lc_p > 1  is.na(lc_p)] = 1
    df = data.frame(logFC <- lc_logFC, `-log10(P)` <- -log10(lc_p), col <- col_vector, gene <- lc_gene)
    colnames(df) <- c('logFC', '-log10(P)', "col", "gene")
    lc_tp_logFC <- df$logFC
    lc_tp_logFC[lc_p>=Threshold_p] = 0
    lc_idx <- order(lc_tp_logFC)[c(lc_rep, length(lc_gene)+1-lc_rep)]
    df$logFC[df$logFC > 10] = 10
    df$logFC[df$logFC < -10] = -10
    res <- ggplot() + geom_point(aes(logFC, `-log10(P)`, col=I(col)), data = df) 
    res <- res + theme_bw() + theme(panel.grid=element_line(colour=NA))
    res <- res + geom_hline(yintercept=-log10(Threshold_p), linetype="longdash")
    res <- res + geom_vline(xintercept=c(Threshold_logFC, -Threshold_logFC), linetype="longdash")
    res <- res + geom_text_repel(data=df[lc_idx,],aes(logFC,`-log10(P)`,label=gene), force=T, max.overlaps=Inf)
    res
}

f_cluster_averages <- function(lc_scRNA, lc_metaN='ident'){
    # 切分出Clusters
    lc_clusters <- SplitObject(lc_scRNA, split.by = lc_metaN)
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- lc_clusters[[lc_i]][[lc_clusters[[lc_i]]@active.assay]]@scale.data
    }
    for (lc_i in 1:length(lc_clusters)){
        lc_clusters[[lc_i]]  <- apply(lc_clusters[[lc_i]],1,mean)
    }
    lc_clusters <- data.frame(lc_clusters)
    scale(lc_clusters)
}
 
library(clusterProfiler)
library(pheatmap)
library(ggdendro)
f_DEG_hclust <- function(lc_counts){
    ggdendrogram(hclust(dist(t(lc_counts))), rotate = T, size = 3)+theme(axis.text = element_text(size=14,face = "bold"))
}

f_DEG_pheatmap_choose_matrix <- function(lc_tp_d, lc_significant_markers, lc_n = 120, Threshold_logFC = 1){
    res <- subset(lc_significant_markers, abs(avg_log2FC) > Threshold_logFC)
    res <- res[order(abs(res$avg_log2FC), decreasing = T),]
    res <- head(unique(res$gene), n = lc_n)
    res <- lc_tp_d[res,]
    res
}

require(ggplotify)
f_DEG_pheatmap <- function(choose_matrix){
    choose_matrix = t(scale(t(choose_matrix)))
    as.ggplot(pheatmap(choose_matrix))
}

f_prepare4CSOmap <- function(lc_scRNA, lc_csomap_data_dir, lc_className){
    lc_csomap_data_dir <- system(paste("echo", lc_csomap_data_dir), intern = T)
    if(!file.exists(lc_csomap_data_dir)){dir.create(lc_csomap_data_dir)}
    # 导出label.txt
    labels <- lc_scRNA[[lc_className]]
    labels$cells <- gsub("-", "." ,rownames(labels)) # TPM的colnames 不知为何导出时被替换了，这里也替换一下
    labels$labels <- as.character(labels[[lc_className]])
    rownames(labels) <- NULL
    labels = labels[,c("cells", "labels")]
    write.table(labels, file.path(lc_csomap_data_dir, "label.txt"), row.names = F, sep = "\t", quote = F) # 不要引号
 
    # copy LR_pairs.txt
    file.copy(from = file.path(lc_csomap_data_dir,"..","demo","LR_pairs.txt"), to = file.path(lc_csomap_data_dir, "LR_pairs.txt"))
    
    # 导出TPM.txt
    tpm <- exp(lc_scRNA[['RNA']]@data)
    tpm <- tpm - 1
    tpm <- tpm*100 # 1E4 to 1E6
    colnames(tpm)[1] = paste0('T', colnames(tpm)[1]) # 预留\t位置
    write.table(tpm, file.path(lc_csomap_data_dir, "TPM.txt"), sep = "\t", quote = F) # 不要引号
    
    lc_fix <- tempfile()
    lc_py <- sprintf('
import mmap, os
 
def mapfile(filename, *args, size=None, **kwargs):
    file = open(filename, *args, **kwargs)
    if size is None: size = os.path.getsize(filename)
    return mmap.mmap(file.fileno(), size)
 
path = "%s"
print(path, "%s")
f = mapfile(path,"r+", size=10)
f[0:1] = b"\t"
print(f[:])
f.close()
print("Done")
', file.path(lc_csomap_data_dir, "TPM.txt"), lc_fix)
    print(lc_py)
    cat(file=lc_fix, lc_py)
    print(system(paste("python3", lc_fix), intern = T))
}

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

```

```
# 配置数据和mark基因表的路径
root_path = "~/zlliu/R_data/hBLA"
 
# 配置结果保存路径
output_path = "~/zlliu/R_data/21.10.04.split"
if (!file.exists(output_path)){dir.create(output_path)}
 
# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()
```

```
# 1、读取数据
scRNA = readRDS("~/zlliu/R_output/21.09.21.SingleR/scRNA.rds")
scRNA <- subset(x = scRNA, !!sym("Region")%in%f_br_cluster_f(scRNA, "Region"))
scRNA@meta.data
```

```
options(repr.plot.width=12, repr.plot.height=6)
options(ggrepel.max.overlaps = Inf)
f_pie_metaN(scRNA, "Region") + f_pie_metaN(scRNA, "hM1_hmca_class")
```

```
n_ExN <- c('L4 IT','L5 IT','L5 ET','IT','L6b','L5/6 IT Car3','L6 IT','L2/3 IT','L5/6 NP','L6 IT Car3','L6 CT')
 
n_InN <- c('Lamp5','Pvalb','Sst','Vip','Sncg')
 
n_NoN <- c('Astro','PAX6','Endo','Micro-PVM','OPC','Oligo','Pericyte','VLMC')
 
n_groups <- list(NoN=n_NoN, ExN=n_ExN, InN=n_InN)
 
f_listUpdateRe <- function(lc_obj, lc_bool, lc_item){
    lc_obj[lc_bool] <- rep(lc_item,times=sum(lc_bool))
    lc_obj
}
 
f_grouplabel <- function(lc_meta.data, lc_groups){
    res <- lc_meta.data[[1]]
    for(lc_g in names(lc_groups)){
        lc_bool = (res %in% lc_groups[[lc_g]])
        for(c_n in colnames(lc_meta.data)){
            lc_bool = lc_bool  (lc_meta.data[[c_n]] %in% lc_groups[[lc_g]])
        }
        res <- f_listUpdateRe(res, lc_bool, lc_g)
    }
    names(res) <- rownames(lc_meta.data)
    res
}
```

```
options(repr.plot.width=9, repr.plot.height=5)
tp_test <- f_br_cluster(scRNA, 'Region', 'hM1_hmca_class')
f_q_frequnency(tp_test)
friedman.test(as.matrix(tp_test))
chisq.test(as.matrix(tp_test), simulate.p.value = TRUE)
fisher.test(as.matrix(tp_test), simulate.p.value = TRUE)
tp_test
options(repr.plot.width=9, repr.plot.height=9)
f_UMAP_more(scRNA, c('hM1_class', 'hmca_class'))
f_UMAP_more(scRNA, c('hM1_hmca_class', 'Region'))
```

```
scRNA[['n_groups']] <- f_grouplabel(scRNA[[c("hM1_hmca_class")]], n_groups)
sc_Neuron <- subset(x = scRNA, n_groups %in% c("InN", "ExN"))
sc_Neuron <- subset(x = sc_Neuron, !!sym("Region")%in%f_br_cluster_f(sc_Neuron, "Region"))
```

```
options(repr.plot.width=12, repr.plot.height=6)
options(ggrepel.max.overlaps = Inf)
f_pie_metaN(sc_Neuron, "Region") + f_pie_metaN(sc_Neuron, "hM1_hmca_class")

options(repr.plot.width=9, repr.plot.height=5)
tp_test <- f_br_cluster(sc_Neuron, 'Region', 'hM1_hmca_class')
f_q_frequnency(tp_test)
friedman.test(as.matrix(tp_test))
chisq.test(as.matrix(tp_test), simulate.p.value = TRUE)
fisher.test(as.matrix(tp_test), simulate.p.value = TRUE)
tp_test

```

```
sc_Neuron_ExN <- subset(x = sc_Neuron, n_groups == "ExN")

options(repr.plot.width=12, repr.plot.height=6)
options(ggrepel.max.overlaps = Inf)
f_pie_metaN(sc_Neuron_ExN, "Region") + f_pie_metaN(sc_Neuron_ExN, "hM1_hmca_class")

options(repr.plot.width=9, repr.plot.height=5)
tp_test <- f_br_cluster(sc_Neuron_ExN, 'Region', 'hM1_hmca_class')
f_q_frequnency(tp_test)
friedman.test(as.matrix(tp_test))
chisq.test(as.matrix(tp_test), simulate.p.value = TRUE)
fisher.test(as.matrix(tp_test), simulate.p.value = TRUE)
tp_test

options(repr.plot.width=9, repr.plot.height=9)
f_UMAP_more(sc_Neuron_ExN, c('hM1_class', 'hmca_class'))
f_UMAP_more(sc_Neuron_ExN, c('hM1_hmca_class', 'Region'))

```

```
sc_Neuron_InN <- subset(x = sc_Neuron, n_groups == "InN")

options(repr.plot.width=12, repr.plot.height=6)
options(ggrepel.max.overlaps = Inf)
f_pie_metaN(sc_Neuron_InN, "Region") + f_pie_metaN(sc_Neuron_InN, "hM1_hmca_class")

options(repr.plot.width=9, repr.plot.height=5)
tp_test <- f_br_cluster(sc_Neuron_InN, 'Region', 'hM1_hmca_class')
f_q_frequnency(tp_test)
friedman.test(as.matrix(tp_test))
chisq.test(as.matrix(tp_test), simulate.p.value = TRUE)
fisher.test(as.matrix(tp_test), simulate.p.value = TRUE)
tp_test

options(repr.plot.width=9, repr.plot.height=9)
f_UMAP_more(sc_Neuron_InN, c('hM1_class', 'hmca_class'))
f_UMAP_more(sc_Neuron_InN, c('hM1_hmca_class', 'Region'))

```

```
Idents(sc_Neuron) <- sc_Neuron[['hM1_hmca_class']]
all_markers <- FindAllMarkers(sc_Neuron, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)

```

```
options(repr.plot.width=12, repr.plot.height=6)
tp_d <- f_cluster_averages(sc_Neuron, "hM1_hmca_class")
tp_d_br <- f_cluster_averages(sc_Neuron, "Region")
f_DEG_hclust(tp_d) + f_DEG_hclust(tp_d_br)
```

```
Idents(sc_Neuron) <- sc_Neuron[['hM1_hmca_class']]
all_markers <- FindAllMarkers(sc_Neuron, min.pct = 0.25, logfc.threshold = 0.25)
significant_markers <- subset(all_markers, subset = p_val_adj<0.05)

```

```
options(repr.plot.width=9, repr.plot.height=30)
p1 <- f_DEG_pheatmap(f_DEG_pheatmap_choose_matrix(tp_d, significant_markers))
options(repr.plot.width=9, repr.plot.height=30)
p2 <- f_DEG_pheatmap(f_DEG_pheatmap_choose_matrix(tp_d_br, significant_markers_br, Threshold_logFC = 0.1))
options(repr.plot.width=18, repr.plot.height=30)
p1 + p2
```

```
f_prepare4cellphoneDB(sc_Neuron,"Neuron", "hM1_hmca_class")
f_prepare4cellphoneDB(sc_Neuron,"Neuron_br", "Region")
f_prepare4CSOmap(sc_Neuron, "~/CSOmap/data/zlliu_s_Neuron", "hM1_hmca_class")
f_prepare4CSOmap(sc_Neuron, "~/CSOmap/data/zlliu_s_Neuron_br", "Region")
```

```
#!/bin/bash
#PBS -q batch
#PBS -V
#PBS -o /home/rqzhang/cellphonedb.out
#PBS -e /home/rqzhang/cellphonedb.err
#PBS -l nodes=1:ppn=32
#PBS -r y
 
cd /home/rqzhang/zlliu/R_data/21.10.04.split/Neuron
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

cd /home/rqzhang/zlliu/R_data/21.10.04.split/Neuron_br
cellphonedb method statistical_analysis  cellphonedb_meta.txt  cellphonedb_count.txt --counts-data=gene_name  --threads=32
cellphonedb plot dot_plot
cellphonedb plot heatmap_plot cellphonedb_meta.txt

```

```
#!/bin/bash
#PBS -q batch
#PBS -V
#PBS -o /home/rqzhang/zlliu/PBS/CSOmap/CSOmap.out
#PBS -e /home/rqzhang/zlliu/PBS/CSOmap/CSOmap.err
#PBS -l nodes=1:ppn=1
#PBS -r y
cd /home/rqzhang/CSOmap/code
matlab -nodisplay -r "runme('zlliu_s_Neuron');exit"
matlab -nodisplay -r "runme('zlliu_s_Neuron_br');exit"
echo $HOME
```

```
n_d <- f_readcellphoneDB('Neuron')
tp_img <- f_cDB_dotplot(subset(n_d$s_m_p, pval<0.05))
f_image_output('Neuron',tp_img, width = 1080,height = 1080)

```

## 更新

```
f_prepare4cellphoneDB_sp <- function(lc_scRNA, lc_dir, lc_className, lc_groupN){
    if (!file.exists(lc_dir)){dir.create(lc_dir)}
    lc_clusters <- SplitObject(lc_scRNA, split.by = lc_groupN)
    for(lc_g in names(lc_clusters)){
        c_dir = file.path(lc_dir,lc_g)
        if (!file.exists(c_dir)){dir.create(c_dir)}
        f_prepare4cellphoneDB(lc_clusters[[lc_g]], c_dir, lc_className)
    }
}

```

```
f_prepare4cellphoneDB_sp(sc_Neuron,"Neuron_br_sp", "hM1_hmca_class", "Region")
```