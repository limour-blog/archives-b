---
title: 通过conda安装纯净环境的Logistic分析包
tags: []
id: '1690'
categories:
  - - 超算基础
date: 2022-02-18 16:00:09
---

[https://xianxiongma.github.io/Clinical-model/chapter2/chapter2.html](https://xianxiongma.github.io/Clinical-model/chapter2/chapter2.html)

*   conda create -n logistic -c conda-forge r-base=4.1.2
*   conda activate logistic
*   conda install -c conda-forge r-irkernel=1.3
*   conda install -c conda-forge r-foreign=0.8\_82
*   conda install -c conda-forge r-mass=7.3\_55
*   conda install -c conda-forge r-hmisc=4.6\_0
*   conda install -c conda-forge r-reshape2=1.4.4