---
title: All与ZIP联用,os.walk的使用
tags:
  - ALL
  - os.walk
  - Python
  - ZIP
id: '130'
categories:
  - - Python练习
date: 2020-05-30 15:35:24
---

```
import re
sep = re.compile(r',\s*')
def _sep(s):
    s = sep.sub(' ', s).strip()
    return s.split(' ')

```

```
import os, re

directory = r'H:\课件\Python'
space = re.compile(r'\s')

def _sum(py):
    temp = (space.sub('', line) for line in py)
    s = sum(line[0] != '#' for line in temp if line)
    #print (s)
    return(s)

s = 0
for p in os.walk(directory):
    for f in p[2]:
        path = os.path.join(p[0],f)
        fname,ext = os.path.splitext(f)
        if ext == '.py':
            #try:
                #with open(path) as py:
                    #print(path)
                    #print(list(py))
                    #s += _sum(py)
            #except UnicodeDecodeError:
                with open(path, encoding='utf-8') as py:
                    s += _sum(py)

```