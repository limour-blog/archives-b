---
title: WGCNA：概念初探
tags: []
id: '1928'
categories:
  - - uncategorized
date: 2022-07-23 17:12:19
---

[组织/细胞的功能执行具有模块化的特点](http://journals.im.ac.cn/html/cjbcn/2017/11/gc17111791.htm)。权重基因共表达网络分析(Weighted gene co-expression network analysis，WGCNA)使用[Pearson相关系数](https://zh.m.wikipedia.org/zh-hans/%E5%9F%BA%E5%9B%A0%E5%85%B1%E8%A1%A8%E8%BE%BE%E7%BD%91%E7%BB%9C)或[bicor双权重中位相关系数](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3586947/)来衡基因之间的共表达关系，将表达模式相似的基因聚类成模块。同一模块的基因可能参与同一生物学过程或通路，被称为功能模块。WGCNA通过分析功能模块与特定性状或表型之间的关联关系，可发现有生物学意义的功能模块。WGCNA除了应用于RNA-Seq数据，也可以用于分析蛋白质组数据、miRNA表达数据，甚至是脑电图数据、大气PM2.5的分析。

## 基本流程

*   WGCNA的基本流程
*   1、[数据归一化](https://www.jianshu.com/p/c515ef000946)，以及[分位数归一化](http://www.bio-info-trainee.com/2043.html)，保证样品间基因表达谱的可比性
*   2、计算基因表达相关矩阵、构建幂指数的邻接矩阵A（邻接矩阵只是相关矩阵逐元素求了β次幂）
*   A是分布在0到1之间的数值组成的对称矩阵，所对应的网络为[无向赋权图](https://zhuanlan.zhihu.com/p/41429668)，是所有后续分析的基础。
*   3、将A被转换为**[拓扑重叠矩阵TOM](https://zhuanlan.zhihu.com/p/441952423)**，以降低噪音和假相关。
*   4、1-TOM得到节点相异度矩阵，对节点相异度矩阵进行聚类分析来鉴定网络模块
*   5、计算模块内基因的连接度，连接度高的基因可能是模块关键基因hub gene
*   6、将模块或关键基因和外部信息进行关联，如临床信息，挖掘出有生物学意义的模块或关键基因。

*   WGCNA的缺点
*   **最少要15个样本才适合此分析**，推荐20个以上的样本。
*   整合其他数据如蛋白质-蛋白质相互作用和甲基化才能提供基因调控信息
*   如果数据来自多个组织或多种条件，组织特异性/条件特异性模块信号可能会被稀释
*   组织中占少数比例的细胞其基因共表达信号可能受其他细胞掩盖
*   不同的数据预处理和分析参数选择也会引起不同的结果
*   样本数越多，得到的结果越好，需要的计算资源也更多

## **相关术语**

*   **[Co-expression network](https://www.biowolf.cn/biodata/WGCNA01.html)：**无向加权基因网络。aij \= cor(xi, xj)β表示unsigned的共表达网络， aij = (1 + cor(xi, xj))/ 2β 表示signed的共表达网络。signed强化了强相关，弱化了弱相关或负相关，使得相关性数值更符合无标度网络特征，更具有生物意义。
*   **Module：**表达高度相关的基因集。在unsigned的共表达网络中，module对应具有高度绝对相关性的基因集。在signed的网络中，module对应正相关的基因基因集。
*   **Module** **Eigengene ME：**给定**模块**的第一主成分。它被认为可以代表给定基因module的基因表达谱。或许可以用UMAP\_1来替换试试？
*   **Module Membership MM：**将该基因的表达量与module eigengene进行相关性分析就可以得到MM值，MM值本质上是一个相关系数，如果基因和某个module的MM值为0，说明二者根本不相关，该基因不属于这个module; 如果MM的绝对值接近1，说明基因与该module相关性很高。
*   **Intramodular connectivity KIM** **：**衡量的是给定基因相对于特定模块的基因是如何连接或共同表达的。模内连接性可以衡量module membership。
*   **Gene significance GS：**将指定基因的表达量与对应的表型数值进行相关性分析，最终的相关系数的值就是GS，GS反映出基因表达量与表型数据的相关性，GS越高表明指定基因与研究表型越相关。
*   **Module significance：**给定module中所有基因的GS平均值。反应了指定module与表型数据的相关性，Module significance越高表明指定module与研究表型越相关。
*   **Eigengene significance：**模块特征（ME）与样本性状的相关性。跟Module significance表明的一样，也是指定module与表型数据的相关性，值越高表明指定module与研究表型越相关。
*   **Connectivity：**在加权共表达网络中，由于每条边代表两个基因间的相关性的大小，对应一个数值，所以一个基因在共表达网络中的Connectivity定义为与该基因相连的所有边的数值之和。另外，根据相连的基因是否和该基因位于同一个module, 又可以将边分为两类，和该基因位于同一个module内，定义为within，位于不同的modules, 定义为out。可根据within的connectivity来确定该module的hub基因。
*   **Hub gene：**表示在共同表达模块内的具有高Connectivity的基因。 

## RNA-Seq数据中的**应用**

*   鉴定高相关的基因module。往往一组表达高度相关的基因具有相似的生物学功能。可通过此方法初步探索lncRNA的功能。
*   鉴定性状高度相关的基因module。与性状高度相关的基因module可进行后续分析，探索其与性状的生物学功能。
*   寻找hub基因。该类应用在早期的lncRNA研究中很热，如果某个module中有lncRNA作为hub基因，可以继续对该lncRNA进行深度探索。 

*   如果样本性状（分组）比较多，WGCNA可以很直观的比较某一组基因在不同分组的表达情况。
*   性状矩阵：用于关联分析的性状必须是**数值型**特征，[如果是分类变量，需要转换为**0-1**矩阵的形式](https://blog.csdn.net/qazplm12_3/article/details/80001327)。
*   对于样本分组为连续变量，WGCNA很直观的表现特定基因module随连续变量的变化情况。

*   可以对每个模块进行三个层次的分析
*   `1`. [功能富集分析](https://occdn.limour.top/2142.html)查看其功能特征是否与研究目的相符；
*   `2`. 模块与性状进行关联分析，找出与关注性状相关度最高的模块；
*   `3`. 模块与样本进行关联分析，找到样品特异高表达的模块。

## 安装补充包

*   [conda activate wgcna](https://occdn.limour.top/2095.html)
*   conda install -c bioconda bioconductor-ggtree -y
*   conda install -c conda-forge r-ape -y
*   \# install.packages("rphylopic")