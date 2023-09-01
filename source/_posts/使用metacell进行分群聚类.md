---
title: 使用metacell进行分群聚类
tags: []
id: '2035'
categories:
  - - uncategorized
date: 2022-10-01 04:18:02
---

## 预处理

```R
f_QC_plot <- function(sce){
    options(repr.plot.width = 12, repr.plot.height = 6)
    print(Seurat::VlnPlot(sce, features = c("nFeature_RNA", "nCount_RNA", "percent.mt"), ncol = 3))
    plot1 <- Seurat::FeatureScatter(sce, feature1 = "nCount_RNA", feature2 = "percent.mt")
    plot2 <- Seurat::FeatureScatter(sce, feature1 = "nCount_RNA", feature2 = "nFeature_RNA")
    plot1 + plot2
}
f_read10x <- function(sce, project='sce'){
    sce <- Seurat::CreateSeuratObject(sce, project = project,
                            min.cells = 3, min.features = 200)
    
    sce[["percent.mt"]] <- Seurat::PercentageFeatureSet(sce, pattern = "^MT-")
    sce[["percent.rp"]] <- Seurat::PercentageFeatureSet(sce, pattern = "^RP[SL]")
    sce <- subset(sce, nFeature_RNA >= quantile(nFeature_RNA, 0.025) 
                  & nFeature_RNA <= quantile(nFeature_RNA, 0.975) 
                  & nCount_RNA >= quantile(nCount_RNA, 0.025) 
                  & nCount_RNA <= quantile(nCount_RNA, 0.975) 
                  & percent.mt <= quantile(percent.mt, 0.975))
    sce <- Seurat::NormalizeData(sce)
    g2m_genes <- Seurat::CaseMatch(search=Seurat::cc.genes$g2m.genes, 
                                   match=rownames(sce))
    s_genes <- Seurat::CaseMatch(search=Seurat::cc.genes$s.genes, 
                                 match=rownames(sce))
    sce <- Seurat::CellCycleScoring(sce, g2m.features=g2m_genes, s.features=s_genes)
    sce$CC.Difference <- sce$S.Score - sce$G2M.Score
    sce <- sce[!grepl(pattern = "(^MT-^RP[SL])",x = rownames(sce)),]
    sce <- Seurat::SCTransform(sce, vst.flavor = "v2",
                           vars.to.regress = c("CC.Difference", "percent.mt", "percent.rp"),
                           verbose = F)
    sce
}
```

```R
sce <- Seurat::Read10X('filtered_feature_bc_matrix')
sce <- f_read10x(sce, project = 'SRX8890106')
```

## 安装补充包

*   [conda activate seurat](https://occdn.limour.top/2358.html)
*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   wget -e "https\_proxy=http://127.0.0.1:20809" https://github.com/tanaylab/metacell/archive/refs/heads/master.zip -O metacell-master.zip
*   devtools::install\_local('metacell-master.zip')
*   conda install -c bioconda bioconductor-singlecellexperiment -y

## seurat转metacell

```R
###### 构建metacell对象
## 初始化
# 设置存放数据的目录
if(!dir.exists("scdb")){dir.create("scdb")}
metacell::scdb_init("scdb", force_reinit=T)
# 设置存放图形的目录
if(!dir.exists("figs")){dir.create("figs")}
metacell::scfigs_init("figs")  
## 提取高变基因
var.genes <- Seurat::VariableFeatures(sce)
var.genes <- structure(rep(1:length(var.genes)), names=var.genes)
var.genes <- metacell::gset_new_gset(sets = var.genes, desc = "seurat variable genes")
metacell::scdb_add_gset("SRX8890106", var.genes)
## 提取counts矩阵
mat <- Seurat::as.SingleCellExperiment(sce)
mat <- metacell::scm_import_sce_to_mat(mat)
metacell::scdb_add_mat("SRX8890106", mat)
```

## 聚类MetaCell

```R
## 构建平衡KNN图
metacell::mcell_add_cgraph_from_mat_bknn(mat_id = "SRX8890106",
                               gset_id = "SRX8890106",
                               graph_id = "SRX8890106_k100",
                               K = 100,
                               dsamp = F)  # 20,000 cells之内不必抽样
## 共聚类
metacell::mcell_coclust_from_graph_resamp(coc_id = "SRX8890106_n1000", graph_id = "SRX8890106_k100", 
                                min_mc_size = 20,  p_resamp = 0.75, n_resamp=1000)
## 生成初级metacell
metacell::mcell_mc_from_coclust_balanced(coc_id = "SRX8890106_n1000", mat_id = "SRX8890106", mc_id = "SRX8890106",
                               K = 20, min_mc_size = 20, alpha = 2)
## 修剪metacell
metacell::mcell_plot_outlier_heatmap(mc_id = "SRX8890106", mat_id = "SRX8890106", T_lfc = 3)
metacell::mcell_mc_split_filt(new_mc_id = "SRX8890106", mc_id = "SRX8890106", mat_id = "SRX8890106", T_lfc = 3, plot_mats = T)
## 2D图展示Cells与MCs
metacell::mc_colorize_default('SRX8890106')
metacell::mcell_mc2d_force_knn(mc2d_id="SRX8890106", mc_id="SRX8890106", graph_id="SRX8890106_k100")
tgconfig::set_param("mcell_mc2d_height", 1000, "metacell")
tgconfig::set_param("mcell_mc2d_width", 1000, "metacell")
metacell::mcell_mc2d_plot(mc2d_id = "SRX8890106")
```

## 导出MetaCell到seurat

```R
mc <- metacell::scdb_mc('SRX8890106')
sce$metacell <- 0
sce$metacell[names(mc@mc)] <- mc@mc
saveRDS(sce@meta.data, 'SRX8890106_meta.rds')
```