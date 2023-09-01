---
title: NCBI-GEO：SRA文件转FASTQ文件
tags: []
id: '2025'
categories:
  - - uncategorized
date: 2022-09-24 09:36:19
---

## 来源

[https://www.jianshu.com/p/8322e00a9f8a](https://www.jianshu.com/p/8322e00a9f8a)

[https://zhuanlan.zhihu.com/p/353530857](https://zhuanlan.zhihu.com/p/353530857)

[https://github.com/rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng)

## 通过conda安装纯净环境的**sra-tools**

*   conda create -n sra\_tools -c bioconda sra-tools
*   conda activate sra\_tools
*   conda install -c conda-forge pigz -y
*   prefetch

*   wget https://github.com/rofl0r/proxychains-ng/releases/download/v4.16/proxychains-ng-4.16.tar.xz
*   tar -xvf proxychains-ng-4.16.tar.xz
*   cd proxychains-ng-4.16
*   ./configure --prefix=$HOME/dev/proxychains4 --sysconfdir=$HOME/etc
*   make && make install
*   make install-config
*   添加正确的代理
*   ~/dev/xray/xray -c ~/etc/xui2.json &
*   ~/dev/proxychains4/bin/proxychains4 -f ~/etc/proxychains.conf curl www.github.com

## 批量下载SRA文件

*   mkdir upload/zl\_liu/sra/GSE172205
*   cd upload/zl\_liu/sra/GSE172205
*   通过 https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=<GSE\_ID> 得到 <SRP\_ID>
*   通过 https://www.ncbi.nlm.nih.gov/Traces/study/?acc=<SRP\_ID> 搜索 <SRP\_ID>
*   下载 **Total** 的 **Accession List**，上传到 upload/zl\_liu/sra/GSE172205 目录下
*    vdb-config -i 设置http代理，网络好也可以不设置
*   prefetch --option-file SRR\_Acc\_List.txt

## 批量转换为FASTQ文件

```sh
#!/bin/sh
#任务名
TASKN=GSE172205
#设置根目录
ROOTDIR=/home/jovyan/upload/zl_liu/sra/GSE172205
#设置CleanData存放目录
CLEAN=$ROOTDIR/$TASKN
 
mkdir $CLEAN
for  file in `cat SRR_Acc_List.txt`
do
echo $file
mkdir $CLEAN/$file
cd $CLEAN/$file
fasterq-dump --split-3 $ROOTDIR/$file -e 16
pigz -p 16 *
done
```

*   nano 11.sh
*   chmod +x 11.sh
*   ./11.sh

## 后续分析

https://occdn.limour.top/1940.html

https://occdn.limour.top/1934.html