---
title: pytesseract测试
tags:
  - Captcha
  - OCR
  - Python
  - tesseract
id: '297'
categories:
  - - Python练习
date: 2020-07-02 19:37:35
---

```
from PIL import Image
#from itertools import cycle
import os, random
import pytesseract
config = "--psm 8 --oem 0 -c tessedit_char_whitelist=abcdefghijklmnopqrstuvwxyz"
def tesOCR(img):
    return pytesseract.image_to_string(img, lang='eng', config=config)


class Fileset(list):
    def __init__(self, name,  ext='', _read=None, root=None):
        if isinstance(name, str)  :
            self.root = os.path.join(root or os.getcwd(), name)
            self.extend(f for f in os.listdir(self.root) if f.endswith(ext))
            self._read = _read
    def __getitem__(self, index):
        if isinstance(index, int):# index是索引
            return os.path.join(self.root, super().__getitem__(index))
        else:# index是切片
            fileset = Fileset(None)
            fileset.root = self.root
            fileset._read = self._read
            fileset.extend(super().__getitem__(index))
            return fileset
    def getFileName(self, index):
        fname, ext = os.path.splitext(super().__getitem__(index))
        return fname
    def __iter__(self):
        return (os.path.join(self.root, f) for f in super().__iter__())
    def __call__(self):
        retn = random.choice(self)
        if self._read: return self._read(retn)
        else: return retn

sample = Fileset('Captcha', '.jpg', Image.open)

```

[![](data:image/png;base64,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)](https://limour.lanzous.com/iqHJfdxripg)

测试用验证码文件

![](https://img-cdn.limour.top/blog_wp/2020/07/微信图片_20200702203134.png)

与百度OCR的对比,第一个是tesseractOCR

```
#tesOCR.py
from PIL import Image
import pytesseract
from io import BytesIO

config = "--psm 8 --oem 0 -c tessedit_char_whitelist=abcdefghijklmnopqrstuvwxyz"

```

```
#baiduOCR.py
from aip import AipOcr
""" 你的 APPID AK SK """
APP_ID = ''
API_KEY = ''
SECRET_KEY = ''

client = AipOcr(APP_ID, API_KEY, SECRET_KEY)
options = {}
options["language_type"] = "ENG"
options["detect_direction"] = "true"
options["detect_language"] = "true"

```

```
#test.py
import os, random
class Fileset(list):
    def __init__(self, name,  ext='', _read=None, root=None):
        if isinstance(name, str)  :
            self.root = os.path.join(root or os.getcwd(), name)
            self.extend(f for f in os.listdir(self.root) if f.endswith(ext))
            self._read = _read
    def __getitem__(self, index):
        if isinstance(index, int):# index是索引
            return os.path.join(self.root, super().__getitem__(index))
        else:# index是切片
            fileset = Fileset(None)
            fileset.root = self.root
            fileset._read = self._read
            fileset.extend(super().__getitem__(index))
            return fileset
    def getFileName(self, index):
        fname, ext = os.path.splitext(super().__getitem__(index))
        return fname
    def __iter__(self):
        return (os.path.join(self.root, f) for f in super().__iter__())
    def __call__(self):
        retn = random.choice(self)
        if self._read: return self._read(retn)
        else: return retn
def fopen(path):
    with open(path, 'rb') as f:
        return f.read()
sample = Fileset('Captcha', '.jpg', fopen)

OCR = input('请选择验证码识别方式(默认为tesseract, 1为百度OCR):')
if not OCR: from tesOCR import tesOCR as OCR
elif OCR == "1" : from baiduOCR import BaiduOCR as OCR
from baiduOCR import BaiduOCR

```