---
title: WGCNA：官方教程学习之网络分析
tags: []
id: '1929'
categories:
  - - uncategorized
date: 2022-07-24 05:30:51
---

[官网](https://horvath.genetics.ucla.edu/html/CoexpressionNetwork/Rpackages/WGCNA/Tutorials/)下载[zipped data sets](https://horvath.genetics.ucla.edu/html/CoexpressionNetwork/Rpackages/WGCNA/Tutorials/FemaleLiver-Data.zip)和[Male data](https://horvath.genetics.ucla.edu/html/CoexpressionNetwork/Rpackages/WGCNA/Tutorials/MaleLiver-Data.zip)，unzip解压，[基本概念见此](https://occdn.limour.top/2144.html)。

## 数据输入与清洗

```R
library(WGCNA)  #加载WGCNA包
enableWGCNAThreads()  #开启多线程
femData = read.csv("LiverFemale3600.csv") #载入基因表达量数据
femData
 
datExpr0 = as.data.frame(t(femData[, -c(1:8)]))  #提取加转置
colnames(datExpr0) = femData$substanceBXH #基因名字
rownames(datExpr0) = colnames(femData)[-c(1:8)]  #样品名字
datExpr0
```

上面的代码可以得到一个列名为基因，行名为样本的表达矩阵，表达量已经归一化。

```R
gsg = goodSamplesGenes(datExpr0, verbose = 3);
gsg$allOK
if (!gsg$allOK)
{
  # Optionally, print the gene and sample names that were removed:
  if (sum(!gsg$goodGenes)>0)
    printFlush(paste("Removing genes:", paste(names(datExpr0)[!gsg$goodGenes], collapse = ", ")));
  if (sum(!gsg$goodSamples)>0)
    printFlush(paste("Removing samples:", paste(rownames(datExpr0)[!gsg$goodSamples], collapse = ", ")));
  # Remove the offending genes and samples from the data:
  datExpr0 = datExpr0[gsg$goodSamples, gsg$goodGenes]
}
```

上面的代码用于删除存在缺失值的基因和样本

```R
# 删除离群样本
sampleTree = hclust(dist(datExpr0), method = "average")
plot(sampleTree, main = "Sample clustering to detect outliers", sub="", xlab="", cex.lab = 1.5,
     cex.axis = 1.5, cex.main = 2)
abline(h = 15, col = "red") #划定需要剪切的枝长
clust = cutreeStatic(sampleTree, cutHeight = 15, minSize = 10)
# 这时候会从高度为15这里横切，把离群样本分开
table(clust)   
keepSamples = (clust==1)  #保留非离群(clust==1)的样本
datExpr = datExpr0[keepSamples, ]  #去除离群值后的数据
```

上面的代码用于删除离群样本

```R
sampleTree = hclust(dist(datExpr0), method = "ward.D2")
p <- ggtree::ggtree(ape::as.phylo(sampleTree),linetype = "dashed",layout = "circular") 
p <- p + ggtree::geom_tiplab()
p 
```

画一个更好看的聚类图

```R
traitData = read.csv("ClinicalTraits.csv");
# 删除我们不需要的数据
allTraits = traitData[, -c(31, 16)];
allTraits = allTraits[, c(2, 11:36) ];  #只保留数值型数据
allTraits
```

读取性状数据

```R
# 将临床表征数据和表达数据进行匹配（用样本名字进行匹配）
f_rm_colN <- function(df, regex){
    df[,!grepl(regex, colnames(df))]
}
traitRows = match(rownames(datExpr), allTraits$Mice)
datTraits = allTraits[traitRows,]
rownames(datTraits) = datTraits$Mice
datTraits <- f_rm_colN(datTraits, 'Mice')
datTraits
collectGarbage() # WGCNA专用GC方法
```

清洗性状数据

```R
sampleTree = hclust(dist(datExpr), method = "average")
traitColors = numbers2colors(datTraits, signed = FALSE) #用颜色代表关联度
plotDendroAndColors(sampleTree, traitColors,
                    groupLabels = colnames(datTraits),
                    main = "Sample dendrogram and trait heatmap")
```

颜色越深，代表这个表型数据与这个样本的基因表达量关系越密切。

## 选择合适的β值

```R
powers = c(c(1:10), seq(from = 12, to=20, by=2))
sft = pickSoftThreshold(datExpr, powerVector = powers, verbose = 5)
plot(sft$fitIndices[,1], -sign(sft$fitIndices[,3])*sft$fitIndices[,2],
     xlab="Soft Threshold (power)",ylab="Scale Free Topology Model Fit,signed R^2",type="n",
     main = paste("Scale independence"));
text(sft$fitIndices[,1], -sign(sft$fitIndices[,3])*sft$fitIndices[,2],
     labels=powers,cex=0.9,col="red");
     abline(h=0.90,col="red")  #查看位于0.9以上的点，可以改变高度值
```

上面的代码用于绘制无标度拓扑拟合指数图，输入的数据为清洗好的表达矩阵。一般选择在0.9以上的，第一个达到0.9以上数值，作为β值。

```R
plot(sft$fitIndices[,1], sft$fitIndices[,5],
     xlab="Soft Threshold (power)",ylab="Mean Connectivity", type="n",
     main = paste("Mean connectivity"))
text(sft$fitIndices[,1], sft$fitIndices[,5], labels=powers, cex=0.9,col="red")
```

β值选接近平缓的第一个值，网络的连通性较好。`sft$powerEstimate`可以获得推荐值

```R
# 无向网络在power小于15或有向网络power小于30内，没有一个power值可以使
# 无标度网络图谱结构R^2达到0.8，平均连接度较高如在100以上，可能是由于
# 部分样品与其他样品差别太大。这可能由批次效应、样品异质性或实验条件对
# 表达影响太大等造成。可以通过绘制样品聚类查看分组信息和有无异常样品。
# 如果这确实是由有意义的生物变化引起的，也可以使用下面的经验power值。
nSamples = nrow(datExpr)
type = "unsigned" # or signed
if (is.na(power)){
  power = ifelse(nSamples<20, ifelse(type == "unsigned", 9, 18),
          ifelse(nSamples<30, ifelse(type == "unsigned", 8, 16),
          ifelse(nSamples<40, ifelse(type == "unsigned", 7, 14),
          ifelse(type == "unsigned", 6, 12))       
          )
          )
}
```

## 计算邻接矩阵

```R
corFnc = "cor" # Pearson correlation
corOptions = list(use = "p") # ?cor了解cor的use参数，此外method参数可以选择"spearman"
corFnc = "bicor"
corOptions = list(maxPOutliers = 0.05, use = "p") # ?bicor了解相关参数
A = adjacency(datExpr, type = "unsigned", power = 6, corFnc = corFnc, corOptions = corOptions); # 6即为β值
```

## 计算**拓扑重叠矩阵**和节点相异度矩阵

```R
TOM = TOMsimilarity(A)
dissTOM = 1-TOM
```

## 相异度矩阵聚类分析以鉴定网络模块

```R
# Call the hierarchical clustering function
geneTree = hclust(as.dist(dissTOM), method = "average");
# Plot the resulting clustering tree (dendrogram)
plot(geneTree, xlab="", sub="", main = "Gene clustering on TOM-based dissimilarity",
labels = FALSE, hang = 0.04);
# We like large modules, so we set the minimum module size relatively high:
minModuleSize = 30;
# Module identification using dynamic tree cut:
dynamicMods = cutreeDynamic(dendro = geneTree, distM = dissTOM,
deepSplit = 2, pamRespectsDendro = FALSE,
minClusterSize = minModuleSize);
table(dynamicMods)
# Convert numeric lables into colors
dynamicColors = labels2colors(dynamicMods)
table(dynamicColors)
# Plot the dendrogram and colors underneath
plotDendroAndColors(geneTree, dynamicColors, "Dynamic Tree Cut",
dendroLabels = FALSE, hang = 0.03,
addGuide = TRUE, guideHang = 0.05,
main = "Gene dendrogram and module colors")
```

```R
# Calculate eigengenes
MEList = moduleEigengenes(datExpr, colors = dynamicColors)
MEs = MEList$eigengenes
# Calculate dissimilarity of module eigengenes
MEDiss = 1-cor(MEs);
# Cluster module eigengenes
METree = hclust(as.dist(MEDiss), method = "average");
# Plot the result
plot(METree, main = "Clustering of module eigengenes",
xlab = "", sub = "")
MEDissThres = 0.25
# Plot the cut line into the dendrogram
abline(h=MEDissThres, col = "red")
# Call an automatic merging function
merge = mergeCloseModules(datExpr, dynamicColors, cutHeight = MEDissThres, verbose = 3)
# The merged module colors
mergedColors = merge$colors;
# Eigengenes of the new merged modules:
mergedMEs = merge$newMEs;
plotDendroAndColors(geneTree, cbind(dynamicColors, mergedColors),
c("Dynamic Tree Cut", "Merged dynamic"),
dendroLabels = FALSE, hang = 0.03,
addGuide = TRUE, guideHang = 0.05)
# Rename to moduleColors
moduleColors = mergedColors
# Construct numerical labels corresponding to the colors
colorOrder = c("grey", standardColors(50));
moduleLabels = match(moduleColors, colorOrder)-1;
MEs = mergedMEs;
```

## 关联网络模块与性状

```R
# 计算相关性和P值
moduleTraitCor = cor(MEs, datTraits, use = "p");
moduleTraitPvalue = corPvalueStudent(moduleTraitCor, nSamples);
```

```R
# 绘图可视化
textMatrix = paste(signif(moduleTraitCor, 2), "\n(",
                   signif(moduleTraitPvalue, 1), ")", sep = "");
labeledHeatmap(Matrix = moduleTraitCor,
               xLabels = names(datTraits),
               yLabels = names(MEs),
               ySymbols = names(MEs),
               colorLabels = FALSE,
               colors = blueWhiteRed(50),
               textMatrix = textMatrix,
               setStdMargins = FALSE,
               cex.text = 0.5,
               zlim = c(-1,1),
               main = paste("Module-trait relationships"))
```

越红的模块越正相关，越蓝的模块越负相关。

### 关联网络模块与基因

```R
geneModuleMembership = as.data.frame(cor(datExpr, MEs, use = "p"));
MMPvalue = as.data.frame(corPvalueStudent(as.matrix(geneModuleMembership), nSamples));
names(geneModuleMembership) = paste("MM", substring(names(MEs), 3), sep="");
names(MMPvalue) = paste("p.MM", substring(names(MEs), 3), sep="");
```

### 关联基因与性状

```R
weight = as.data.frame(datTraits$weight_g);
geneTraitSignificance = as.data.frame(cor(datExpr, weight, use = "p"));#和体重性状的关联
GSPvalue = as.data.frame(corPvalueStudent(as.matrix(geneTraitSignificance), nSamples));
names(geneTraitSignificance) = paste("GS.", names(weight), sep="");
names(GSPvalue) = paste("p.GS.", names(weight), sep="");
```

### 可视化基因与模块、性状的相关性

```R
# 运行以下代码可视化GS和MM
module = "blue"
column = match(module, substring(names(MEs), 3));
moduleGenes = moduleColors==module;
par(mfrow = c(1,1));
verboseScatterplot(abs(geneModuleMembership[moduleGenes, column]),
                   abs(geneTraitSignificance[moduleGenes, 1]),
                   xlab = paste("Module Membership in", module, "module"),
                   ylab = "Gene significance for body weight",
                   main = paste("Module membership vs. gene significance\n"),
                   cex.main = 1.2, cex.lab = 1.2, cex.axis = 1.2, col = module)
```

MM-GS图中的每一个点代表一个基因，横坐标值表示基因与模块的相关性，纵坐标值表示基因与表型性状的相关性，与性状高度显著相关的基因往往是与这个性状显著相关的模块中的重要元素。

## 获得基因注释结果

```R
# 基本注释框架
annot = read.csv(file = "GeneAnnotation.csv");
probes2annot = match(names(datExpr), annot$substanceBXH);
sum(is.na(probes2annot)) # 为0，即全匹配上了。
geneInfo0 = data.frame(substanceBXH = names(datExpr),
                       geneSymbol = annot$gene_symbol[probes2annot],
                       LocusLinkID = annot$LocusLinkID[probes2annot],
                       moduleColor = moduleColors,
                       geneTraitSignificance,
                       GSPvalue);
# 添加模块成员的信息
modOrder = order(-abs(cor(MEs, weight, use = "p"))); # 按照与体重的显著水平将模块进行排序
for (mod in 1:ncol(geneModuleMembership)){
  oldNames = names(geneInfo0)
  geneInfo0 = data.frame(geneInfo0, geneModuleMembership[, modOrder[mod]],
                         MMPvalue[, modOrder[mod]]);
  names(geneInfo0) = c(oldNames, paste("MM.", modNames[modOrder[mod]], sep=""),
                       paste("p.MM.", modNames[modOrder[mod]], sep=""))
}
# 相关性排序
geneOrder = order(geneInfo0$moduleColor, -abs(geneInfo0$GS.datTraits.weight_g));
geneInfo = geneInfo0[geneOrder, ]
geneInfo
```

上述基因可以按模块拆分，然后进行[富集分析](https://occdn.limour.top/2128.html)

## 可视化

### 基因网络

*   TOMplot(dissTOM^7, geneTree, moduleColors, main = "Network heatmap plot, all genes")

```R
select = moduleGenes # 某个模块内的基因
selectTOM = dissTOM[select, select];
selectTree = hclust(as.dist(selectTOM), method = "average")
selectColors = moduleColors[select];
TOMplot(selectTOM^7, selectTree, selectColors, main = paste0("Network heatmap plot, ", module, ' module gene'))
```

### 表征基因网络

```R
names(weight) = "weight"
MET = orderMEs(cbind(MEs, weight))
plotEigengeneNetworks(MET, "", marDendro = c(0,4,1,2), marHeatmap = c(3,4,1,2), cex.lab = 0.8, xLabelsAngle
                      = 90)
plotEigengeneNetworks(MET, "Eigengene dendrogram", marDendro = c(0,4,2,0),
                      plotHeatmaps = FALSE)
plotEigengeneNetworks(MET, "Eigengene adjacency heatmap", marHeatmap = c(3,4,2,2),
                      plotDendrograms = FALSE, xLabelsAngle = 90)
```

从分层聚类图有一个叶子是weight，从中可以看出weight与哪些模块更接近

## 导出网络

[Cytoscape](https://cytoscape.org/)源自系统生物学，用于将生物分子交互网络与高通量基因表达数据和其他的分子状态信息整合在一起，可以用于大规模蛋白质-蛋白质相互作用、蛋白质-DNA和遗传交互作用的[分析](https://zhuanlan.zhihu.com/p/220527695)。

```R
modules = c("blue", "magenta");
inModule = is.finite(match(moduleColors, modules));
modTOM = TOM[inModule, inModule];
modProbes = names(datExpr)[inModule];
dimnames(modTOM) = list(modProbes, modProbes)
modGenes = annot$gene_symbol[match(modProbes, annot$substanceBXH)];
cyt = exportNetworkToCytoscape(modTOM,
                               edgeFile = paste("CytoscapeInput-edges-", paste(modules, collapse="-"), ".txt", sep=""),
                               nodeFile = paste("CytoscapeInput-nodes-", paste(modules, collapse="-"), ".txt", sep=""),
                               weighted = TRUE,
                               threshold = 0.02,
                               nodeNames = modProbes,
                               altNodeNames = modGenes,
                               nodeAttr = moduleColors[inModule])
```