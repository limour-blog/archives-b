---
title: 复旦高性能计算 (三) jupyter R内核 png设备问题的解决方法
tags: []
id: '652'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-09-19 22:32:20
---

*   第一步 安装 [`Cairo包`](https://cran.r-project.org/web/packages/Cairo/index.html)
*   第二步 `.Rprofile` 文件添加以下内容

```
options(bitmapType='cairo')
```

*   第三步 `~/.local/share/jupyter/kernels/ir/kernel.json`

```
{
  "argv": ["/home/rqzhang/zlliu/dev/R/R_s/lib64/R/bin/R", "--slave", "-e", "IRkernel::main()", "--args", "{connection_file}"],
  "display_name": "R",
  "language": "R"
}
```