---
title: 复旦高性能计算 (六) jupyter启用R内核
tags: []
id: '1098'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-10-22 19:10:23
---

## 第一步 启用镜像

```
install.packages("BiocManager", repos="http://mirrors.tuna.tsinghua.edu.cn/CRAN/" )
options(BioC_mirror="https://mirrors.tuna.tsinghua.edu.cn/bioconductor")
```

## 第二步 安装依赖

```
install.packages(c('repr', 'IRdisplay', 'evaluate', 'crayon', 'pbdZMQ', 'devtools', 'uuid', 'digest', 'cairo', 'IRkernel'), repos="http://mirrors.tuna.tsinghua.edu.cn/CRAN/" )
```

## 第三步 启用R内核

```
IRkernel::installspec(user = FALSE)
```

## 第四步 解决图片输出问题

[复旦高性能计算 (三) jupyter R内核 png设备问题的解决方法](https://limour.top/652.html)