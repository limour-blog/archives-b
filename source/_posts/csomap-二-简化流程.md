---
title: CSOmap (二) 简化流程
tags: []
id: '856'
categories:
  - - 单细胞下游分析
  - - 生物信息学
date: 2021-10-03 00:12:52
---

```
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
matlab -nodisplay -r "runme('zlliu_Neuron');exit"
matlab -nodisplay -r "runme('zlliu_Neuron_br');exit"
echo $HOME
```