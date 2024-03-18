---
title: 为Docker内的jupyter开启代码提示
tags:
  - Docker
  - jupyter
  - nbextensions
id: '548'
categories:
  - - Raspberry Pi
date: 2021-03-08 14:45:49
---

```
sudo docker exec -it jupyter /bin/bash
pip3 install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
pip3 install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com  jupyter_nbextensions_configurator
jupyter nbextensions_configurator enable --user
kill 1
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/03/image.webp)](https://img.limour.top/archives_2023/blog_wp/2021/03/image.webp)

取消disable，勾选Hinterland