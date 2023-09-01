---
title: 原神 AI全自动钓鱼 使用方法
tags: []
id: '1308'
categories:
  - - 有趣技能
date: 2021-12-15 21:03:45
---

## 代码来源

[https://github.com/7eu7d7/genshin\_auto\_fish](https://github.com/7eu7d7/genshin_auto_fish)

## 第一步 安装 Anaconda

*   [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)
*   [Getting started with Anaconda](https://www.anaconda.com/products/individual#Downloads)
*   下载安装包，安装到独立的磁盘，取消掉两个Advanced Options

[![](https://img-cdn.limour.top/blog_wp/2021/12/image.png)](https://img-cdn.limour.top/blog_wp/2021/12/image.png)

## 第二步 安装cuda和cuDNN

*   [win10安装CUDA和cuDNN的正确姿势](https://zhuanlan.zhihu.com/p/94220564?utm_source=wechat_session&ivk_sa=1024320u)
*   下载或更新最新的 [GEFORCE EXPERIENCE](https://www.nvidia.cn/geforce/geforce-experience/download/)
*   使用 GEFORCE EXPERIENCE 更新最新驱动
*   桌面右键打开英伟达控制面板，点击帮助->系统信息->组件->NVCUDA.DLL 获取cuda版本
*   [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_local)
*   [NVIDIA Developer Program](https://developer.nvidia.com/rdp/cudnn-download)
*   按教程安装，覆盖，设置好环境变量
*   cuda安装时取消除cuda外的其他选项

[![](https://img-cdn.limour.top/blog_wp/2021/12/image-1.png)](https://img-cdn.limour.top/blog_wp/2021/12/image-1.png)

## 第三步 修改 conda 镜像

*   conda config --set show\_channel\_urls yes
*   conda config --show
*   conda config --add pkgs\_dirs G:\\python\\conda\\pkgs
*   conda config --add envs\_dirs G:\\python\\conda\\envs
*   conda config --remove pkgs\_dirs C:\\Users\\11248.conda\\pkgs
*   conda config --remove pkgs\_dirs C:\\Users\\11248\\AppData\\Local\\conda\\conda\\pkgs
*   conda config --remove envs\_dirs C:\\Users\\11248.conda\\envs
*   conda config --remove envs\_dirs C:\\Users\\11248\\AppData\\Local\\conda\\conda\\envs
*   [Anaconda 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/) 添加镜像
*   conda clean -i

## 第四步 修改 pip 镜像

*   [pip安装使用国内镜像](https://codeantenna.com/a/tx5U9UvUFu)
*   创建 pip/pip.ini
*   （linux下，修改 ~/.pip/pip.conf）
*   写入以下内容

```ini
[global] 

index-url = http://pypi.douban.com/simple

trusted-host = pypi.douban.com

disable-pip-version-check = true

pip timeout = 120
```

```txt
# 豆瓣
https://pypi.doubanio.com/simple/
# 阿里云    
https://mirrors.aliyun.com/pypi/simple/
# 清华大学
https://pypi.tuna.tsinghua.edu.cn/simple/
https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/
```

## 第4.5步 老老实实安装 visual-cpp-build-tools

*   [https://visualstudio.microsoft.com/visual-cpp-build-tools/](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
*   点上C++桌面开发的勾，mmp，绕不过

## 第五步 安装 [genshin\_auto\_fish](https://github.com/7eu7d7/genshin_auto_fish)

*   git clone https://github.com/7eu7d7/genshin\_auto\_fish.git
*   conda create -n ysfish python=3.6
*   conda activate ysfish
*   conda prompt 打开 genshin\_auto\_fish 目录
*   cd G:\\github\\genshin\_auto\_fish
*   python -m pip install -U pip --user
*   下载 [pycocotools\_windows-2.0.0.2-cp36-cp36m-win\_amd64.whl](https://pypi.tuna.tsinghua.edu.cn/packages/a4/22/77b6374592c04c10a1c57ff069e1bcd435af8021b64ff94bd958bfcc6c10/pycocotools_windows-2.0.0.2-cp36-cp36m-win_amd64.whl#sha256=fbafce7a9abbdc6003cf8e29ca28ce970c5f8ec202fd63233459ff9c51f502e8)
*   pip install .\\pycocotools\_windows-2.0.0.2-cp36-cp36m-win\_amd64.whl
*   删除 requirements.py 里的 git+https://github.com/philferriere/cocoapi.git#subdirectory=PythonAPI
*   python requirements.py --cuda 110 --proxy http://127.0.0.1:10808
*   python requirements.py --cuda 110

## 第5.5步 修改一些文件

*   .\\yolox\\utils\\metric.py 36行 max\_mem = int(total \* mem\_ratio) - 1000
*   .\\yolox\_tools\\train.py torch.set\_num\_threads(2)
*   yolox\_tiny\_fish.py self.data\_num\_workers = 0 self.max\_epoch = 9
*   python yolox\_tools/train.py -f yolox/exp/yolox\_tiny\_fish.py -d 1 -b 8 --fp16 -c weights/yolox\_tiny.pth --cache
*   .\\fishing.py 152 env = Fishing(delay=0.1, max\_step=10000, show\_det=False)
*   .\\fisher\\environment.py 205 if np.mean(np.abs(self.img\[py,px,:\]-self.std\_color))>15:
*   .\\fisher\\environment.py 122 self.std\_color=np.array(\[255, 250, 190\])
*   .\\fishing.py 150 def start\_fishing(predictor, agent, bite\_timeout=120):
*   .\\fishing.py 174 times=-1 175 break 176 if times == -1: continue
*   .\\fishing.py 183 time = time.time() 189 if done or time.time() - time > 120:

## 第六步 按Readme操作即可