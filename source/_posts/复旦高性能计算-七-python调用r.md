---
title: 复旦高性能计算 (七) Python调用R
tags: []
id: '1102'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-10-22 20:47:24
---

## 前置要求

*   [复旦高性能计算 (一) Linux无root权限安装软件：以R为示例](https://limour.top/631.html) 这里编译时开启了--enable-R-shlib
*   [复旦高性能计算 (六) jupyter启用R内核](https://limour.top/1098.html) 确保不出奇怪的问题
*   install.packages(c('RJSONIO', 'httr'), repos="http://mirrors.tuna.tsinghua.edu.cn/CRAN/" )

## 第一步 安装rpy2

```
pip3.8 install rpy2 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

*   查看rpy2的安装路径

```
import sys
sys.path
```

*   配置环境变量 每次启动先运行

```
import os
os.environ['R_HOME'] = '/home/rqzhang/zlliu/dev/R/R_s/lib64/R' # R的安装后含SVN-REVISION的目录 非bin目录
os.environ['R_USER'] = '/home/rqzhang/zlliu/dev/python38/lib/python3.8/site-packages/rpy2' # rpy2的安装目录
```

[冷门编译好的python库](http://www.lfd.uci.edu/~gohlke/pythonlibs)

## 第二步 调用方法一（不推荐）

```
from rpy2.robjects import r as Rcode
from rpy2.robjects.packages import importr as Rrequire
Rrequire('ggplot2') # 导入R包
print(Rcode("pi")) # 运行R语句

#rx()相当于”[“操作（注意取出的是R的list对象），而rx2()相当于”[[“操作
```

[pandas.py: Comparison with R / R libraries](https://pandas.pydata.org/pandas-docs/stable/getting_started/comparison/comparison_with_r.html)

## 第三步 调用方法二（推荐）

*   开启 %R 支持

```
%load_ext rpy2.ipython
```

*   测试数据集

```
import pandas as pd
data = pd.DataFrame({'x':[1,2,3,4,5,6],'y':[1,2,3,4,5,6]})
px = 1
py = 2
```

*   cell内调用R

```
%%R -i data -o colN
require(ggplot2)
colN = colnames(data)
ggplot(data=data) + geom_point(aes(x=x,y=y))
```

*   一行内调用R

```
%R require(ggplot2)
%R -i px,py
z = %R px*py
print(colN, z[0]
```

[如何优雅地在Jupyter Notebook中同时运行R和IPython](https://zhuanlan.zhihu.com/p/139032905)