---
title: loom文件使用记录
tags: []
id: '2043'
categories:
  - - uncategorized
date: 2022-10-03 15:51:46
---

随着单细胞数据量的增长，计算要求成指数增长，当数据量大于10万个细胞的时候，seurat包分析就显得非常有压力了，因为在实时内存中储存数据就变得非常困难，HDF5数据格式提供了高效的磁盘存储，而不是在内存中存储数据，这就将分析扩展到大规模数据集，甚至可以达到大于100万细胞的级别 ，Linnarson实验室开发了一种基于hdf5的数据结构，loom，可以方便地存储单细胞基因组数据集和元数据。他们发布了一个名为loompy的Python API来与loom文件交互，而[loomR](https://satijalab.org/loomr/loomr_tutorial)能基于R的与loom交互（[Merlin\_cd6c](https://www.jianshu.com/p/7067e0ec6ed8)）

## 安装补充包

*   [conda activate seurat](https://occdn.limour.top/2371.html)
*   conda install -c conda-forge r-hdf5r -y
*   \# conda install -c bioconda r-loom=0.2.0.2 -y
*   wget https://github.com/mojaveazure/loomR/archive/refs/heads/develop.zip -O loomR-develop.zip
*   devtools::install\_local('loomR-develop.zip')
*   conda install -c conda-forge binutils\_impl\_linux-64 -y
*   BiocManager::install("hdf5r")

*   conda create -n loom -c conda-forge loompy=3.0.6 -y
*   conda activate loom

## 下载数据

[The Human Cell Atlas](https://www.humancellatlas.org/) is an international collaborative consortium that charts the cell types in the healthy body, across time from development to adulthood, and eventually to old age. This enormous undertaking, larger even than the Human Genome Project, will transform our understanding of the 37.2 trillion cells in the human body.

[The HCA Data Portal](https://data.humancellatlas.org/) stores and provides single-cell data contributed by labs around the world. Anyone can contribute data, find data, or access community tools and applications.

*   `wget 'https://storage.googleapis.com/datarepo-a6e6a252-bucket/8b504553-0300-426e-876c-116772d06c6e/485f373c-f3b3-47a2-8f9b-bbe007a4a75b/ProstateCellAtlas-human-prostate-gland-10xv2.loom?X-Goog-Algorithm=GOOG4-RSA-SHA256&X-Goog-Credential=datarepo-jade-api%40terra-datarepo-production.iam.gserviceaccount.com%2F20221003%2Fauto%2Fstorage%2Fgoog4_request&X-Goog-Date=20221003T121317Z&X-Goog-Expires=900&X-Goog-SignedHeaders=host&X-Goog-Signature=7eab3ff712b55cf9812cc24b379500175005b7dc5e11e11d35cb0e36522310cf6c46574fc606754eb54a57286ac0fd60d6bd7d5871a1692b80be993c0ae96d85252df7acaf3367acd1e7f2dfdc41abbcfc0abb6305fa00c9ede58e843840e93685c5e47567b8dbcf6563411d294ae1ddc244f6434a5901ee4955f75df8993a4e39417b73357ac240d5fcb2d1f4699554480d3eccc8aec594e9b8b3511f164290559f4788e7b2ea5a3b3bd6aee21ebdfbe032859d27671b496480d29d9df5bafd790bf411e10da377f112c64c49900c4f9ce9a4566e87b327ba2550530a71969fd8e0fa0d637b481d496a50c86aea8dc010348e00bb39f6a69ce38ab70b51c77e' -O ProstateCellAtlas-human-prostate-gland-10xv2.loom`

类似的网站：[CZ CELLxGENE](https://cellxgene.cziscience.com/)；可以[检索需要的数据集](https://cellxgene.cziscience.com/datasets)，下载rds格式的文件

*   `curl -o local.rds "https://corpora-data-prod.s3.amazonaws.com/5484f890-7e6f-4eb4-821f-2fc995d10c47/local.rds?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIATLYQ5N5X7IE7U3FW%2F20221003%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221003T131441Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEOz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLXdlc3QtMiJHMEUCIDqVa2igyV9%2BOaaQGXPAxZb%2FrSKXPH0JNBf1zPb5S33wAiEA1zFz8ZKqLcGz9Ip%2FqeJvw0JMTKkptGz%2FInBtdfurxZEq9AMIpf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgwyMzE0MjY4NDY1NzUiDOTNFg85zD2F0B0FISrIA0CT3GUFmKR4ScNFtM%2B0FU4nIVrD7lM%2FecCE9SVHJM%2F6QGWPA7E3LWr9KcWMByy3jEQ9FRAjlsr26v2d8Znt2tAkvIcF7xiQaR1qgISi0Xak0EZceNUMTknU85CCPwPC7%2FiwxckmUX%2B0o0V11yh3rQyhHvpdcgO5COGafKwMIHYtldJNuTF0B9TTf1WVJkfZv%2Bl6o7DxHahwkb9oWWBKON9cM37oXDcH7m%2BD6szQamWY6bKNN2QNcBkKFy66ndKeFpdz6annSJKR98c8MmwGdoA9DzwixxiXZhIOR7IGD9N5oIy28sd9ZyPnBwSOEmkHB%2B61TXkwV28Li7NFfe7z6a0kpG3LiRSKPmhtC8cu5LlFUnLKqHeD1tXrVkOixukdPacWLQyAQDSYaUGoZVS7ESW2Y11bSxZLCEZ97RIM5OwOQIfv%2FsQUQLic9qDmlStQCYAVBkdmApA4a3kKbMHcOVOwECwTnUDEr36oevJMqaP688egzOjpfqkshXGLRZ%2BtA9yIn1RGqrwT%2Bl%2BStPc2s2hFBue6q9sd1aVmS3VDfiRHlvMgDS3et0jnWvUVih95U8RaeMBNVYfl4P%2BMMbhgmPFEp%2B%2F03nS%2BbzD9ouuZBjqlAYt4%2FOil7fZYzwhlUd0nHWnBLsP5w0g1EBURMhQHCtKBcNf2deWPQLlnQFQSwZhCOaYtNhZ679%2FDcsiCUCgQJUj4sTm1kbxjQouHVaOnml%2FlYM6VaJo6aLjvB%2F%2Bx9u4G925PZK1CUToWyoVyWw4sPhbGuBYRTPzzNIACi5ZuCB0DZ0cjhD86pv71dJIE%2BfwzGlEzzyVEK5nkSuKbxw4aYBOA3oTaXA%3D%3D&X-Amz-Signature=0e5c2ff76b231c5591672b93de4a77128e5757e50c699636a757e8eefba180aa"`
*   mv local.rds cellxgene\_Human\_prostate.rds

## 使用记录

公共的集群是真难用，一堆包装半天装不上，磁盘IO慢的一批。。。。。

```R
conda activate seurat
.libPaths('')
sce <- loomR::connect(filename = "./HumanCellAtlas/ProstateCellAtlas/ProstateCellAtlas-human-prostate-gland-10xv2.loom", mode = "r", skip.validate = T)
mat <- sce[["matrix"]][,]
gene <- sce$row.attrs$Gene[]
# barcode <- sce$col.attrs$CellID[]
barcode <- sce$col.attrs$cell_names[]
mat <- t(mat)
colnames(mat)= barcode
rownames(mat)= gene
sce$close_all()
sce <- Seurat::CreateSeuratObject(counts = mat, project = 'prostate', min.cells = 3, min.features = 200)
rm(mat)
gc()
#             used   (Mb)  gc trigger     (Mb)    max used     (Mb)
# Ncells   3611333  192.9     6136950    327.8     6136950    327.8
# Vcells 287451723 2193.1 45703419996 348689.5 57126196661 435838.3
sce
# An object of class Seurat 
# 39879 features across 128673 samples within 1 assay 
# Active assay: RNA (39879 features, 0 variable features)
```