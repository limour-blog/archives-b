---
title: 基于PaddleOCR的PDF转WORD
tags: []
id: '2188'
categories:
  - - uncategorized
date: 2023-04-11 13:57:30
---

[PP-Structure](https://github.com/PaddlePaddle/PaddleOCR)是PaddleOCR团队自研的智能文档分析系统，旨在帮助开发者更好的完成版面分析、表格识别等文档理解相关任务。

PP-Structurev2的主要特性如下：

*   支持对图片/pdf形式的文档进行版面分析，可以划分**文字、标题、表格、图片、公式等**区域；
*   支持通用的中英文**表格检测**任务；
*   支持表格区域进行结构化识别，最终结果输出**Excel文件**；
*   支持基于多模态的关键信息抽取(Key Information Extraction，KIE)任务-**语义实体识别**(Semantic Entity Recognition，SER)和**关系抽取**(Relation Extraction，RE)；
*   支持**版面复原**，即恢复为与原始图像布局一致的word或者pdf格式的文件；
*   支持自定义训练及python whl包调用等多种推理部署方式，简单易用；
*   与半自动数据标注工具PPOCRLabel打通，支持版面分析、表格识别、SER三种任务的标注。

## 安装下载工具

*   [文件加速代下载服务](https://occdn.limour.top/2561.html)，[GitHub Proxy](https://ghproxy.com/)
*   安装[aria2](https://github.com/aria2/aria2/releases)，并添加到环境变量Path，
*   安装 Aria2 Explorer [chrome浏览器拓展](https://chrome.google.com/webstore/detail/aria2-explorer/mpkodccbngfoacfalldjimigbofkhgjn)
*   可以 aria2c --enable-rpc 配合 Aria2 Explorer 进行下载

## 安装conda

*   下载miniconda：[清华镜像](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)
*   conda config --set show\_channel\_urls yes
*   notepad.exe $env:HOMEPATH.condarc 确保是 [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)

## 安装cuda

*   下载或更新最新的 [GEFORCE EXPERIENCE](https://www.nvidia.cn/geforce/geforce-experience/download/)
*   桌面右键打开英伟达控制面板，点击帮助->系统信息->组件->NVCUDA.DLL 获取cuda版本
*   下面的步骤可能不需要，因为conda可以自动安装，只是记录一下
*   查看[cuda](https://zhuanlan.zhihu.com/p/94220564?utm_source=wechat_session&ivk_sa=1024320u)版本：nvcc --version
*   [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)，cuda安装时取消除cuda外的其他选项，并检查环境变量Path里是否有相应的路径
*   [NVIDIA Developer Program](https://developer.nvidia.com/rdp/cudnn-download)，下载cuDNN，解压cuDNN压缩包，可以看到bin、include、lib目录，将其拖拽到cuda目录下的相应版本的根目录，覆盖相应的bin、include、lib目录
*   cd $env:CUDA\_PATH\\extras\\demo\_suite 执行 .\\bandwidthTest.exe

![](https://img-cdn.limour.top/blog_wp/2021/12/image-1.png)

## 安装**[PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)**

*   conda create -n PP -c conda-forge python=3.8
*   conda activate PP
*   conda info --env
*   （二选一 GPU）conda install paddlepaddle-gpu==2.4.2 cudatoolkit=11.6 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/Paddle/ -c conda-forge
*   （二选一 CPU）python -m pip install paddlepaddle -i https://mirror.baidu.com/pypi/simple
*   使用 `python` 进入 python 解释器，输入`import paddle` ，再输入`paddle.utils.run_check()`
*   python -m pip install "paddleocr>=2.6" -i https://mirror.baidu.com/pypi/simple
*   Invoke-WebRequest -Uri "https://ghproxy.com/https://raw.githubusercontent.com/PaddlePaddle/PaddleOCR/release/2.6/ppstructure/recovery/requirements.txt" -OutFile "requirements.txt"
*   python -m pip install -r requirements.txt -i https://mirror.baidu.com/pypi/simple
*   python -m pip install "PyMuPDF==1.18.7" -i https://mirror.baidu.com/pypi/simple （解决[Issue#877](https://github.com/pymupdf/PyMuPDF/issues/877)）

## PDF转WORD

*   准备一份没有嵌字，纯扫描件的UnrealText.pdf
*   paddleocr --image\_dir=UnrealText.pdf --type=structure --recovery=true
*   效果比直接用Acrobat好一点
*   如果是简短的一段文字，还是直接用[Umi-OCR](https://github.com/hiroi-sora/Umi-OCR/releases)识别图片方便一点（基于PaddleOCR）
*   等Microsoft 365 Copilot正式出来后，对paddleocr重建的docx进行智能纠错和格式美化应该效果会好一点。