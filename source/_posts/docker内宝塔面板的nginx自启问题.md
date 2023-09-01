---
title: Docker内宝塔面板的NGINX自启问题
tags: []
id: '1920'
categories:
  - - uncategorized
date: 2022-07-20 03:07:49
---

之前在树莓派里[通过docker安装了宝塔面板](https://occdn.limour.top/2020.html)，并将[博客转移到了树莓派上](https://occdn.limour.top/2024.html)，通过[内网穿透](https://occdn.limour.top/1999.html)和[反向代理](https://occdn.limour.top/1997.html#%E7%AC%AC%E4%B8%89%E6%AD%A5-docker%E5%AE%89%E8%A3%85NPM%E9%9D%A2%E6%9D%BF)提供服务。今天家里停电了一会儿，后来来电后发现访问博客出现502错误，原来是宝塔内部的NGINX没有自动启动，下面尝试解决这个问题。

*   sudo docker inspect baota 发现入口是用python3启动了一个script.py
*   sudo docker exec -it baota /bin/bash 发现进入了/app这个目录，且目录下有script.py文件
*   通过宝塔自带的文件编辑功能，在script.py文件里的main函数中的循环前添加了下面的代码
*   通过手动停止再启动容器，发现NGINX成功自启，问题解决\*★,°\*:.☆(￣▽￣)/$:\*.°★\* 。

```python
    try:
        os.system('/etc/init.d/nginx start')
    except:
        pass
```