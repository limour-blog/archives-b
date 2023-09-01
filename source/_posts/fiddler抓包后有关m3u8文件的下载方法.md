---
title: Fiddler抓包后有关M3U8文件的下载方法
tags:
  - Fiddler
  - m3u8
  - pyhon
id: '509'
categories:
  - - Python练习
date: 2021-01-21 19:30:09
---

Fiddler抓包结果完整保存为TXT文件前面部分内容如下：

```
GET http://api.readoor.cn/manager/Data/getMediaM3u8/1.m3u8?X-APPID=1544059443&X-AT=2.0&X-VER=2.70.1&X-TK=f7b71f35cacf94c404ba9335a8f9adfc&X-SEQ=25&X-DT=kJjts2qMTm5c7T3X1BBGQ50Dsqu9a3bHVoqS6Z7%252BxiLyQzLTVSS0CgOKpEKStZQsSh65qQ8IvsiUwg4JpOQLMPfIOweN%252Ffk5cYl%252FPq1x2Xc%252BM31JBDoYhbZ2qGHDZcNRFAWnqBySIGsd9%252FvQ6ZPMOdhHqLhfMwyEvVDs%252Fk0baqLjgy26jOJSUznhDFcW13uJqogSgDjkex173NNBqaqU6aQPgm5UUcjB%252F5afPcigKHN2xmCs6t9uVCn5evgySGQO1lN%252FlTz0DpDnjC4m9I2UJ7mb10Nu20oGA%252BNYcY6rk%252BZOXr8Dq0jFIHUXPxThm9CpO5WeoPo6OfuVxfxobQD4UvmcMSRRh3EegkMmt5N9sC3luRvd1s6yDDr0I8GJe57M3GbeEzL%252FtsbOZSS5N9ZGB9U5pZGkoqZc7PqBirbRLpP6%252FN29kOAKwnQQSDH7E3EIshGx&X-CT=2.0&X-AS=519977e4ae11b01ef63e511afa0688d0&X-F=72977&X-OS=2&X-TZ=GMT%2B08%3A00&X-UID=9559629&X-MK=1&X-UT=1&X-TS=1611223926516&X-NO=d0834db4023deea85c95ae2112f88157 HTTP/1.1
User-Agent: Mozilla/5.0 (Linux; Android 10; YAL-AL00 Build/HUAWEIYAL-AL00; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/87.0.4280.101 Mobile Safari/537.36
Accept-Encoding: gzip
Host: api.readoor.cn
Connection: Keep-Alive


HTTP/1.1 200 OK
Date: Thu, 21 Jan 2021 10:12:06 GMT
Content-Type: application/vnd.apple.mpegurl
Connection: keep-alive
X-Powered-By: PHP/5.6.40
Set-Cookie: csrf_cookie_name=; expires=Thu, 21-Jan-2021 12:12:05 GMT; Max-Age=7200; path=/
Set-Cookie: vpapp_session=; expires=Thu, 21-Jan-2021 12:12:05 GMT; Max-Age=7200; path=/; HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Access-Control-Allow-Origin: *
Content-Length: 8688

```

通过下述代码下载：

```
from Crypto.Cipher import AES
import requests, os, re, time, traceback
from requests.packages.urllib3.exceptions import InsecureRequestWarning
from hyper.contrib import HTTP20Adapter
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
from tqdm import tqdm
from itertools import count
import re
ncols = 80

def _path(name, root=None):
    p =  os.path.join(root or os.getcwd(), name)
    return (os.path.exists(p) or not os.mkdir(p)) and p

def getf(id):
    with open(f'{id:02}.txt') as f:
        return f.read().split('\n\n')

_ua = re.compile(r'\nUser-Agent: (.*)\n')
_ac = re.compile(r'\nAccept-Encoding: (.*)\n')
_cn = re.compile(r'csrf_cookie_name=(.+?)\b')
_es = re.compile(r'expires=(.+?);')
_vs = re.compile(r'vpapp_session=(.+?)\b')
def getHeader(a, b):
    c_n = _cn.findall(b)[0]
    e_s = _es.findall(b)[0]
    v_s = _vs.findall(b)[0]
    header = {
        'user-agent': _ua.findall(a)[0],
        'accept-encoding': _ac.findall(a)[0],
        'cookie': f'csrf_cookie_name={c_n}; expires={e_s}; vpapp_session={v_s}; HttpOnly'
    }
    return header

_ur = re.compile(r'URI="(.+?)"')
def getKey(c, header):
    s =  requests.session()
    s.verify = False
    s.mount(r'https://', HTTP20Adapter())
    URL = _ur.findall(c)[0]
    r = s.get(URL, headers=header)
    return r.content

_iv = re.compile(r',IV=(.+?)\b')
def getIV(c):
    return _iv.findall(c)[0]

_ts = re.compile(r'\nhttps://(.+?).ts')
def getTsList(c):
    res = _ts.findall(c)
    return (f'https://{x}.ts' for x in res)

def rqGet(URL, header):
    s = requests.session()
    s.verify = False
    s.mount(r'https://', HTTP20Adapter())
    r = s.get(URL, headers=header)
    return r.content

def getTsData(KEY, IV, DATA, METHOD='AES-128'):
    if METHOD == 'AES-128':
        #url_key = "key.key"  # 省略写法
        #content_key = requests.get(url_key).content  # 获取key值
        #vi = "0x00000000000000000000000000000000"  # vi值一样
        vt = IV.replace("0x", "")[:16].encode()  # 偏移码
        ci = AES.new(KEY, AES.MODE_CBC, vt)  # 构建解码器
        #content = requests.get(url_content_ts).content  # 获取加密视频内容
        content_ts = ci.decrypt(DATA)  # 解码ts中的二进制格式
        return content_ts

def dl(id):
    a,b,c = getf(id)
    header = getHeader(a, b)
    key = getKey(c, header)
    iv = getIV(c)
    tsl = getTsList(c)
    path = _path(f'{id:02}')
    for i,URL in enumerate(tqdm(tsl, ncols=ncols, desc=f'Dling...{id:02}...')):
        while True:
            try:
                d = rqGet(URL, header)
                d = getTsData(key, iv, d)
                p = os.path.join(path, f'{i:03}')
                with open(p, 'wb') as f:
                    f.write(d)
                    break
            except:
                traceback.print_exc()
                continue

```

通过以下方式合并ts文件：

```
copy /b  * movie_new.ts
```