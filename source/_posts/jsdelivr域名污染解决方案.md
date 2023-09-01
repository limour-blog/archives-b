---
title: jsDelivr域名污染解决方案
tags: []
id: '1824'
categories:
  - - 建站探索
date: 2022-05-16 22:48:07
---

## 第一步 反代JSD

```nginx
#PROXY-START/

location ^~ /
{
    expires 30d;
    valid_referers limour.top *.limour.top;
    if ($invalid_referer){
        return 403;
    }
    gzip_proxied any;
    proxy_pass https://jscdn.limour.top;
    proxy_set_header Host jscdn.limour.top;
    proxy_ssl_server_name on;
    proxy_ssl_name jscdn.limour.top;
    resolver 8.8.8.8;
    sub_filter_once off;
    sub_filter_types *;
    sub_filter "jscdn.limour.top" "jscdn.limour.top";
    
}

#PROXY-END/
```

```shell
sed -i "s/jscdn.limour.top/jscdn.limour.top/g" `grep -rl 'jscdn.limour.top' Sakura`
```

## 第二步 修正所有链接

```python
# -*- coding:utf-8 -*-
# @Author:zgd
# @time:2019/6/21
# @File:isBinaryFile.py
 
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
 
    fileDir = r"E:\apktool\base"
 
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
            line = line.replace(r'jscdn.limour.top',r'jscdn.limour.top')
            f.write(line)    
 
        f.close()  
 
if __name__=='__main__':
 
    main() 

```