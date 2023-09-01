---
title: R安装包前的一些配置
tags: []
id: '1292'
categories:
  - - 超算基础
date: 2021-12-07 18:56:30
---

## 第一步 创建.Rprofile 写入以下内容

```
.libPaths("/home/jovyan/Rlibrary")
options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
options(CRAN="http://mirrors.tuna.tsinghua.edu.cn/CRAN/")
options(repr.plot.width=12, repr.plot.height=12)
options(ggrepel.max.overlaps = Inf)
Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS=TRUE)
Sys.setenv(http_proxy="http://127.0.0.1:20809")
```

## 第二步 给git设置代理

```
git config --global http.proxy 'socks5://127.0.0.1:20808'
git config --global https.proxy 'socks5://127.0.0.1:20808'
```

## 第三步 安装常用包管理

```
install.packages('BiocManager')
install.packages('devtools')
```

## 第四步 一些常用的安装方法

*   R CMD

```
export C_INCLUDE_PATH=/opt/conda/include/freetype2
export CPLUS_INCLUDE_PATH=/opt/conda/include/freetype2

wget https://download.savannah.gnu.org/releases/freetype/freetype-2.11.1.tar.gz
R CMD INSTALL systemfonts_1.0.3.tar.gz
```

*   conda 安装依赖

```
proxychains4 conda install jags
```

*   conda 本地安装

```
conda install --use-local boost-1.59.0-py27_0.tar.bz2
```

## 第五步 编辑.profile，连接时启动代理

```shell
PIDS=`ps -ef grep 'xray run -c' grep -v grep  awk '{print $2}'`
if [ "$PIDS" != "" ]; then
echo "xray is runing!"
else
xray run -c $HOME/etc/xui2.json >$HOME/log/xray.log 2>&1 &
fi
```