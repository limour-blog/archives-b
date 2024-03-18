---
title: CSOmap (一) 分析流程
tags: []
id: '710'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-09-21 14:43:08
---

## 第一步 明确CSOmap所需的数据

*   先读readme，里面描述了输入数据的格式

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-17.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-17.webp)

*   label.txt：细胞标注表，格式如图

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-13.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-13.webp)

*   LR\_pairs.txt：受体配体表，格式如图，用CSOmap自带的也行

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-11.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-11.webp)

*   TPM.txt：TPM标注化的表达矩阵，格式如图，第一行得用`\t`开头

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-18.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-18.webp)

## 第二步 导出CSOmap所需的数据

```
# 0、加载程辑包
library(Seurat)
library(dplyr)
library(patchwork)

# 配置数据路径
root_path = "~/zlliu/R_data/21.07.15.IntegrateData"

# 配置结果保存路径
output_path = "~/CSOmap/data/zlliu_IntegrateData_region"
if (!file.exists(output_path)){dir.create(output_path)}

# 设置工作目录，输出文件将保存在此目录下
setwd(output_path)
getwd()

# 1、读取数据
sn = readRDS(file.path(root_path, 'hBLA_scRNAs.rds'))

# 合并对象
scRNA <- merge(sn$BA213, y=c(sn$BA04, sn$BA06, sn$BA09, sn$BA21,
                                   sn$BA22, sn$BA37, sn$BA39, sn$BA40,
                                   sn$BA41, sn$BA42, sn$BA44, sn$BA45))

# 保存脑区信息
scRNA[["Region"]] <- Idents(scRNA)

# 导出label.txt
labels <- scRNA[["Region"]]
labels$cells <- gsub("-", "." ,rownames(labels)) # TPM的colnames 不知为何导出时被替换了，这里也替换一下
labels$labels <- as.character(labels$Region)
rownames(labels) <- NULL
labels = labels[,c("cells", "labels")]
write.table(labels, "label.txt", row.names = F, sep = "\t", quote = F) # 不要引号

```

*   对TPM.txt的格式进行小修补：

```
import mmap, os

def mapfile(filename, *args, size=None, **kwargs):
    file = open(filename, *args, **kwargs)
    if size is None: size = os.path.getsize(filename)
    return mmap.mmap(file.fileno(), size)

f = mapfile("data/zlliu_V1C/TPM.txt",'r+', size=10)

f[0:1] = b"\t"

```

## 第三步 激活fat节点的matlab

*   [如何在无法连接 Internet 的电脑上进行产品的安装和激活?](https://ww2.mathworks.cn/matlabcentral/answers/130613-internet)
*   [What is a Host ID? How do I find my Host ID in order to activate my license?](https://ww2.mathworks.cn/matlabcentral/answers/101892-what-is-a-host-id-how-do-i-find-my-host-id-in-order-to-activate-my-license)
*   运行 `matlab -nojvm`

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-14.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-14.webp)

*   hostid中选一个， 比如 `6c:92:bf:ee:06:d6`
*   [获得离线许可证](https://ww2.mathworks.cn/licensecenter)

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-15.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-15.webp)

*   将许可证放入用户目录的License path，并修改后缀为 `.lic`
*   测试：`matlab **-nodisplay**`

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-16.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-16.webp)

## 第四步 测试CSOmap分析

```
cd /home/rqzhang/CSOmap/code
matlab -nodisplay
runme('demo');
```

## 第五步 进行CSOmap分析

*   直接分析，不推荐，断网气死：

```
runme('zlliu_IntegrateData_region');
```

*   创建 `21.09.21.CSOmap.pbs`

```
#!/bin/bash
#PBS -q batch
#PBS -V
#PBS -o /home/rqzhang/zlliu/PBS/CSOmap/CSOmap.out
#PBS -e /home/rqzhang/zlliu/PBS/CSOmap/CSOmap.err
#PBS -l nodes=1:ppn=8
#PBS -r y
cd /home/rqzhang/CSOmap/code
matlab -nodisplay -r "runme('zlliu_IntegrateData_region');exit"
echo $HOME
```

*   提交任务：`qsub /home/rqzhang/zlliu/PBS/CSOmap/21.09.21.CSOmap.pbs`