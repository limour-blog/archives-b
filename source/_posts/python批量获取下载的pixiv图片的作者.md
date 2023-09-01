---
title: Python批量获取下载的Pixiv图片的作者
tags:
  - Pixiv
  - Python
id: '445'
categories:
  - - Python练习
  - - 软件分享
date: 2020-11-26 00:17:44
---

经评论区的提醒，决定为本站所有用到的图片添加艺术家的版权信息。

奈何当初只保存了图片，怎么办呢？以下是解决办法：

```
import os, requests, re
#from lxml import etree

proxy = r'http://127.0.0.1:10809'
# proxy = r'http://your_user:your_password@your_proxy_url:your_proxy_port'
proxies = {
    'http': proxy,
    'https': proxy
    }

def N2Id(name):
    Id = name[:8]
    print(Id)
    return Id

def getHtml(Id):
    url = f'https://www.pixiv.net/artworks/{Id}'
    return requests.get(url, proxies=proxies , verify=False)

Aid = re.compile(r'"authorId":"(\d+?)"')
Ana = re.compile(r'"userName":"(.+?)"')
def Id2U(Id): 
    #html = etree.HTML(getHtml(Id).text)
    text = getHtml(Id).text
    st = text.find(f'{Id}_p0.jpg')
    if st == -1: st = text.find(f'{Id}_p0.png')
    if st == -1: print(text)
    text = text[st:st+500]
##    print(text)
##    print(Aid.findall(text))
    authorId = Aid.findall(text)[0]
    userName = Ana.findall(text)[0]
    return f'<a href="https://www.pixiv.net/users/{authorId}" target="_blank" rel="nofollow">{userName}</a>'

Us = (Id2U(N2Id(name)) for name in os.listdir(r'./gallary'))

```