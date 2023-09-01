---
title: cellchat (一) 细胞间通讯分析
tags: []
id: '1070'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-10-15 02:00:03
---

## 教程

[https://github.com/sqjin/CellChat](https://github.com/sqjin/CellChat)

## 第一步 分离比较对象并创建cellchat对象

```r
library(CellChat)
library(ggplot2)                  
library(patchwork)
library(igraph)

HSPC <- subset(sce, group == 'HSPC')
CRPC <- subset(sce, group == 'CRPC')
Idents(HSPC) <- HSPC[['cell_sub_type']]
HSPC <- createCellChat(object = HSPC)
Idents(CRPC) <- CRPC[['cell_sub_type']]
CRPC <- createCellChat(object = CRPC)

DB_SS <- subsetDB(CellChatDB.human, search = "Secreted Signaling")
DB_ER <- subsetDB(CellChatDB.human, search = "ECM-Receptor")
DB_CC <- subsetDB(CellChatDB.human, search = "Cell-Cell Contact")

SS_H <- HSPC
SS_H@DB <- DB_SS
ER_H <- HSPC
ER_H@DB <- DB_ER
CC_H <- HSPC
CC_H@DB <- DB_CC

SS_C <- CRPC
SS_C@DB <- DB_SS
ER_C <- CRPC
ER_C@DB <- DB_ER
CC_C <- CRPC
CC_C@DB <- DB_CC

SS_H <- subsetData(SS_H)
ER_H <- subsetData(ER_H)
CC_H <- subsetData(CC_H)

SS_C <- subsetData(SS_C)
ER_C <- subsetData(ER_C)
CC_C <- subsetData(CC_C)

f_CC_Preprocessing <- function(cellchat){
    cellchat <- identifyOverExpressedGenes(cellchat)
    cellchat <- identifyOverExpressedInteractions(cellchat)
    cellchat <- projectData(cellchat, PPI.human)
    cellchat
}

SS_H <- f_CC_Preprocessing(SS_H)
ER_H <- f_CC_Preprocessing(ER_H)
CC_H <- f_CC_Preprocessing(CC_H)

SS_C <- f_CC_Preprocessing(SS_C)
ER_C <- f_CC_Preprocessing(ER_C)
CC_C <- f_CC_Preprocessing(CC_C)

saveRDS(SS_H, file = 'SS_H.rds')
saveRDS(ER_H, file = 'ER_H.rds')
saveRDS(CC_H, file = 'CC_H.rds')

saveRDS(SS_C, file = 'SS_C.rds')
saveRDS(ER_C, file = 'ER_C.rds')
saveRDS(CC_C, file = 'CC_C.rds')
```

## 第二步 手动进行并行计算

```r
# 公共部分
library(CellChat)
library(ggplot2)                  
library(patchwork)
library(igraph)

future::plan("sequential")

f_CC_Inference_CC_C_network <- function(cellchat){
    cellchat <- computeCommunProb(cellchat, type = "truncatedMean", trim = 0.1, population.size = TRUE)
    cellchat <- filterCommunication(cellchat, min.cells = 10)
    cellchat <- computeCommunProbPathway(cellchat)
    cellchat <- aggregateNet(cellchat)
    cellchat
}

#创建三个文件，分别运行以下代码
#01
MP_H = readRDS(file = 'SS_H.rds')
MP_C = readRDS(file = 'SS_C.rds')

#02
MP_H = readRDS(file = 'ER_H.rds')
MP_C = readRDS(file = 'ER_C.rds')

#03
MP_H = readRDS(file = 'CC_H.rds')
MP_C = readRDS(file = 'CC_C.rds')

#公共部分
MP_H <- f_CC_Inference_CC_C_network(MP_H)
MP_C <- f_CC_Inference_CC_C_network(MP_C)

require(NMF)
require(ggalluvial)
MP_H <- netAnalysis_computeCentrality(MP_H, slot.name = "netP")
MP_C <- netAnalysis_computeCentrality(MP_C, slot.name = "netP")

selectK(MP_H, pattern = "outgoing")
MP_H <- identifyCommunicationPatterns(MP_H, pattern = "outgoing", k = 3)
selectK(MP_H, pattern = "incoming")
MP_H <- identifyCommunicationPatterns(MP_H, pattern = "incoming", k = 4)
selectK(MP_C, pattern = "outgoing")
MP_C <- identifyCommunicationPatterns(MP_C, pattern = "outgoing", k = 3)
selectK(MP_C, pattern = "incoming")
MP_C <- identifyCommunicationPatterns(MP_C, pattern = "incoming", k = 4)

f_CC_ISG_F = function(cellchat){
    cellchat <- computeNetSimilarity(cellchat, type = "functional")
    cellchat <- netEmbedding(cellchat, type = "functional")
    cellchat <- netClustering(cellchat, type = "functional")
    cellchat
}

MP_H <- f_CC_ISG_F(MP_H)
MP_C <- f_CC_ISG_F(MP_C)

f_CC_ISG_S = function(cellchat){
    cellchat <- computeNetSimilarity(cellchat, type = "structural")
    cellchat <- netEmbedding(cellchat, type = "structural")
    cellchat <- netClustering(cellchat, type = "structural")
    cellchat
}

MP_H <- f_CC_ISG_S(MP_H)
MP_C <- f_CC_ISG_S(MP_C)

# 分别保存
saveRDS(MP_H,file = 'CC_H.rds')
saveRDS(MP_C,file = 'CC_C.rds')

saveRDS(MP_H,file = 'ER_H.rds')
saveRDS(MP_C,file = 'ER_C.rds')

saveRDS(MP_H,file = 'SS_H.rds')
saveRDS(MP_C,file = 'SS_C.rds')
```