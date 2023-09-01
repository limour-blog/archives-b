---
title: Windows通过docker安装jupyter-lab
tags:
  - Docker
  - jupyter-lab
  - Windows
id: '580'
categories:
  - - 计算机相关
date: 2021-03-14 14:42:04
---

```
docker run -d \
    --name jupyter \
    --restart unless-stopped \
    -p 18888:8888 \
    -v $HOME:/root/host_home \
--workdir /root \
    leanderd/single-cell-analysis:210114 \
jupyter-lab --no-browser --ip=0.0.0.0 --allow-root /root/host_home --NotebookApp.token=''
```