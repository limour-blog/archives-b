---
title: 量子化学软件Gaussian开启多核
tags:
  - Gaussian
  - IR
id: '111'
categories:
  - - 软件分享
date: 2020-04-26 18:34:14
---

![](https://img.limour.top/archives_2023/blog_wp/2020/04/qrcode_for_gh_2f2011f8d30c_1280-e1587098787293.jpg)

发送 Gauss软件 获得技术支持

第一步 将高斯的路径(一般为`C:\G09W`) 添加的环境变量Path中  
成功标志为Win+X 选择打开Shell 输入g09有信息显示

第二步 在高斯路径下的Scratch文件夹内(`C:\G09W\Scratch`)建立文件  
文件名为`Default.Rou`,内容为`-P- 4`表示开启4线程进行运算  
由于我的是32位,故`-M- 1GB`可以正常运行,过大会报错,可不加  
参数为一行一个

* * *

打开GaussView画出2,3-二甲基-2-丁烯,右键选择`Calculte-Gaussian Calculationa Setup` 选择Job Type中的Energy,其他默认  
`submit-save-submit`进行计算,完成后问关闭窗口-是,打开的文件选`.chk`

再次计算,这次`Job Type`选`Opt+Freq`,Method第二项选DFT,进行计算  
结果文件右键,`Results-Vibrations`可以看到计算出的红外光谱