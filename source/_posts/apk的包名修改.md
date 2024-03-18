---
title: APK的包名修改
tags: []
id: '1222'
categories:
  - - 有趣技能
  - - 计算机相关
date: 2021-11-03 17:41:02
---

## 来源

[apk的包名修改](https://www.cnblogs.com/tianxiaozz/archive/2012/12/26/change_apk_package_name.html)@[tianxiaozz](https://www.cnblogs.com/tianxiaozz/)  
[Apktool重打包Apk详细介绍](https://blog.csdn.net/u010889616/article/details/78198822)  
[Android修改应用包名和ApplicationId：实战和理解](https://www.jianshu.com/p/7643fe0967c1)

## 第一步 安装依赖

*   JDK：[安装JDK@廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1280507291631649)
*   Apktool：[Installation for Apktool](https://ibotpeaches.github.io/Apktool/install/)

```powershell
pip3 install python-magic
pip3 install python-magic-bin
```

## 第二步 逆向解包APK

```powershell
.\apktool.bat d .\xpushdemo.apk
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/0f25ad37e69eea0f05ffdd4ba6cbbab.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/0f25ad37e69eea0f05ffdd4ba6cbbab.webp)

## 第三步 使用python批量替换

*   该文件保存为 `isBinaryFile.py` 用于判断文件是否为文本型文件

```python
# -*- coding:utf-8 -*-
# @Author:zgd
# @time:2019/6/21
# @File:operateSystem.py

import magic, os, stat
import re
import codecs
 
def is_binary_file_1(file_path):
    '''
    根据text文件数据类型判断是否是二进制文件
    :param ff: 文件名（含路径）
    :return: True或False，返回是否是二进制文件
    '''
    TEXT_BOMS = (
        codecs.BOM_UTF16_BE,
        codecs.BOM_UTF16_LE,
        codecs.BOM_UTF32_BE,
        codecs.BOM_UTF32_LE,
        codecs.BOM_UTF8,
    )
    with open(file_path, 'rb') as file:
        CHUNKSIZE = 8192
        initial_bytes = file.read(CHUNKSIZE)
        file.close()
    #: BOMs to indicate that a file is a text file even if it contains zero bytes.
    return not any(initial_bytes.startswith(bom) for bom in TEXT_BOMS) and b'\0' in initial_bytes
 
 
def is_binary_file_2(ff):
    '''
    根据magic文件的魔术判断是否是二进制文件
    :param ff: 文件名（含路径）
    :return: True或False，返回是否是二进制文件
    '''
    mime_kw = 'x-executablex-sharedliboctet-streamx-object'  ###可执行文件、链接库、动态流、对象
    try:
        magic_mime = magic.from_file(ff, mime=True)
        magic_hit = re.search(mime_kw, magic_mime, re.I)
        if magic_hit:
            return True
        else:
            return False
    except Exception as e:
        return False


def is_ELFfile(filepath):
    if not os.path.exists(filepath):
        logger.info('file path {} doesnot exits'.format(filepath))
        return False
    # 文件可能被损坏，捕捉异常
    try:
        FileStates = os.stat(filepath)
        FileMode = FileStates[stat.ST_MODE]
        if not stat.S_ISREG(FileMode) or stat.S_ISLNK(FileMode):  # 如果文件既不是普通文件也不是链接文件
            return False
        with open(filepath, 'rb') as f:
            header = (bytearray(f.read(4))[1:4]).decode(encoding="utf-8")
            # logger.info("header is {}".format(header))
            if header in ["ELF"]:
                # print header
                return True
    except UnicodeDecodeError as e:
        # logger.info("is_ELFfile UnicodeDecodeError {}".format(filepath))
        # logger.info(str(e))
        pass
 
    return False

def is_binary_file(filepath):
    return any((is_binary_file_1(filepath), is_binary_file_2(filepath), is_ELFfile(filepath)))
```

*   运行该文件 `replacetxt.py` 用于替换包名 和 包路径

```python
#!/usr/bin/python
 
import os
 
import re

from isBinaryFile import is_binary_file
#list files
 
def listFiles(dirPath):
 
    fileList=[]
 
    for root,dirs,files in os.walk(dirPath):
 
        for fileObj in files:
 
            fileList.append(os.path.join(root,fileObj))
 
    return fileList
 
def main():
 
    fileDir = r"E:\apktool\xpushdemo"
 
    regex = r'FUNC_SYS_ADD_ACCDETAIL'
 
    fileList = listFiles(fileDir)
 
    for fileObj in fileList:
        if is_binary_file(fileObj):
            continue
        try:
            f = open(fileObj,'r+',encoding='utf-8')
            all_the_lines=f.readlines()
        except:
            f.close()
            continue
 
        f.seek(0)
 
        f.truncate()
 
        for line in all_the_lines:
            line = line.replace(r'com.xuexiang.pushdemo',r'com.limour.pushdemo')
            line = line.replace(r'com/xuexiang/pushdemo',r'com/limour/pushdemo')
            f.write(line)   
 
        f.close()  
 
if __name__=='__main__':
    main() 
```

*   如有 `provider` 的 `authorities` 未更改，自行修改.py文件进行批量修改
*   找到所有这种层级的 `com/xuexiang/pushdemo` 文件夹  
    把对应的文件夹名称更改为包名对应的名称 `com/limour/pushdemo`
*   用 Notepad++ 手动替换 解包目录下 `AndroidManifest.xml` 内的 `com.xuexiang.pushdemo` 为 `com.limour.pushdemo`

## 第四步 打包apk

```powershell
.\apktool.bat b .\xpushdemo
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/image-3.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/image-3.webp)

新打包的apk在解包文件夹的 `dist` 目录下

*   出现 \\res\\values\\xxx.xml 111 `xx.xx.xx` 找不到时，就手动将 `.` 改成 `_`

## 第五步 生成证书

```powershell
keytool -genkey -alias abc.keystore -keyalg RSA -validity 20000 -keystore abc.keystore
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/9abeda08b2b408d196779f058d37a9b.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/9abeda08b2b408d196779f058d37a9b.webp)

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/5e2d83c8f998a1e3b69761778c0e5cc.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/5e2d83c8f998a1e3b69761778c0e5cc.webp)

## 第六步 给apk签名

*   新打包的apk在解包文件夹的 `dist` 目录下，先移动到证书目录，再运行下面的命令

```powershell
jarsigner -verbose -keystore abc.keystore -signedjar xpushdemo_signed.apk xpushdemo.apk abc.keystore
```

[![](https://img.limour.top/archives_2023/blog_wp/2021/11/53bbb0390bb68b8704845cabf3edf0f.webp)](https://img.limour.top/archives_2023/blog_wp/2021/11/53bbb0390bb68b8704845cabf3edf0f.webp)

*   验证签名有效

```powershell
jarsigner -verify -verbose -certs xpushdemo_signed.apk 
```

## 第七步 优化apk （可选，需安装 Android SDK）

*   [Android Studio && SDK](https://developer.android.com/studio)
*   将以下路径添加到环境变量Path  
    C:\\Users\\11248\\AppData\\Local\\Android\\Sdk\\build-tools\\31.0.0  
    C:\\Users\\11248\\AppData\\Local\\Android\\Sdk\\tools\\bin  
    C:\\Users\\11248\\AppData\\Local\\Android\\Sdk\\platform-tools

```powershell
zipalign -v 4 xpushdemo_signed.apk xpushdemo_latest.apk
```