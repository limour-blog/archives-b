---
title: QP (二) 实时荧光定量PCR结果分析
tags: []
id: '1178'
categories:
  - - 基础医学研究
date: 2021-10-30 04:55:11
---

## 前置背景

[QP (一) 实时荧光定量PCR结果导出](https://limour.top/1153.html)

## 第一步 查看误差

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-19.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-19.webp)

误差超过1说明数据有问题，误差0.5以内最好

## 第二步 计算

[【简述】相对定量法分析qRT-PCR数据](https://zhuanlan.zhihu.com/p/32746600)  
[相对定量： ΔΔCT法和相对标准曲线法](https://wenku.baidu.com/view/1021822a0b4e767f5acfceac?ivk_sa=1023194j&bfetype=new)

*   第一步 同一样品的目的基因与内参对应

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-20.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-20.webp)

*   第二步 目的基因的Ct减去内参的平均Ct作为ΔCt

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-21.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-21.webp)

*   第三步 计算ΔΔCt

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-22.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-22.webp)

如果没有对照组，随便选一组当对照，用对照的一个复孔当起始标准

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-23.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-23.webp)

*   第四步 计算倍率变化

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-24.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-24.webp)

## 第三步 绘图

[https://limour.lanzouv.com/i37hI00d1bof](https://limour.lanzouv.com/i37hI00d1bof)

*   第一步 导入数据

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-25.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-25.webp)

*   第二步 绘制柱状图

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/image-26.webp)](https://img.limour.top/archives_2023/blog_wp/2021/10/image-26.webp)