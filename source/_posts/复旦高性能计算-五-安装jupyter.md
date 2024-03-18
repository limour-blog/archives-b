---
title: 复旦高性能计算 (五) 安装jupyter
tags: []
id: '738'
categories:
  - - 生物信息学
  - - 超算基础
date: 2021-09-23 04:59:48
---

## 第一步 安装jupyter模块

```
pip3.8 install jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple
ln -s /home/rqzhang/zlliu/dev/python38/bin/jupyter /home/rqzhang/zlliu/bin/jupyter38
```

## 第二步 开启代码提示

```
pip3.8 install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com jupyter_contrib_nbextensions
jupyter38 contrib nbextension install --user
pip3.8 install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com  jupyter_nbextensions_configurator
jupyter38 nbextensions_configurator enable --user
pip3.8 install nbconvert==5.6.1 -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
```

## 第三步 测试运行

```
/home/rqzhang/zlliu/bin/frpc -c /home/rqzhang/zlliu/frp/frpc.ini &
jupyter38 notebook --ip='0.0.0.0' --port=18888 --no-browser --NotebookApp.token="<token>" --NotebookNotary.db_file=':memory:' --NotebookApp.kernel_manager_class=notebook.services.kernels.kernelmanager.AsyncMappingKernelManager
```

## 结果展示

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-21.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-21.webp)

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-22.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-22.webp)

[![](https://img.limour.top/archives_2023/blog_wp/2021/09/image-19.webp)](https://img.limour.top/archives_2023/blog_wp/2021/09/image-19.webp)