---
title: ImageJ (一) 细胞计数
tags: []
id: '1091'
categories:
  - - ImageJ
date: 2021-10-19 04:50:59
---

## 第二步 图片亮度均匀化

*   PS-钢笔-选区-调整亮度
*   **Image-Type-8-bit**

## 第二步 细胞分割

*   **Image-Adjust-Threshold**
*   **Process-Binary-Fill Holes**
*   **Process-Binary-Watershed**

## 第三步 细胞计数

*   **Analyze-Analyze Particles**