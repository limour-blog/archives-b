---
title: raspios下载
tags:
  - OS
  - Python
  - raspberrypi
  - 异步IO
  - 断点续传
id: '132'
categories:
  - - Raspberry Pi
date: 2020-05-30 15:47:15
---

第一步,从[官网](https://www.raspberrypi.org/downloads/raspbian/)获得下载地址  
第二步,进入AriaNg添加下载:  
`http://downloads.raspberrypi.org/raspios_full_armhf/images/raspios_full_armhf-2020-05-28/2020-05-27-raspios-buster-full-armhf.zip`  
第三步,从Nextcloud获得下载地址和header

```
import asyncio, os
from aiohttp import ClientSession
from tqdm import tqdm
ncols = 80

# proxy = 'http://your_user:your_password@your_proxy_url:your_proxy_port'

_ssl = None
#_ssl = {'verify_ssl':False}
if not _ssl:
    import ssl
    FORCED_CIPHERS = (
    'ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+HIGH:'
    'DH+HIGH:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+HIGH:RSA+3DES'
    )
    sslcontext = ssl.create_default_context()
    sslcontext.options = ssl.OP_NO_SSLv3
    sslcontext.options = ssl.OP_NO_SSLv2
    sslcontext.options = ssl.OP_NO_TLSv1_1
    sslcontext.options = ssl.OP_NO_TLSv1_2
    sslcontext.options = ssl.OP_NO_TLSv1_3
    sslcontext.set_ciphers(FORCED_CIPHERS)
    _ssl= {'ssl':sslcontext}

if os.name == 'nt':
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

async def _dl2f(req, p, block_size, pbar, m='ab'):
    chunk = await req.content.read(block_size)
    with _fopen(p, m) as file:
        while chunk:
            pbar.update(file.write(chunk))
            chunk = await req.content.read(block_size)

async def _dl(url, session, proxy, start, end, p, block_size, pbar):
    if start > end:
        return
    async with session.get(url, headers={'Range': f"bytes={start}-{end}"}, proxy=proxy , **_ssl) as req:
        await _dl2f(req, p, block_size, pbar)

class _pbar(object):
    def __init__(self, dec, unit='B'):
        self.dec = dec
        self.total = 0
        self.unit = unit
    def update(self, size):
        self.total += size
        print(f'{self.dec}: {self.total}{self.unit}', end='\r', flush=True)

def _len(Response):
    #print (Response.headers)
    return int(Response.headers.get('content-length',-1))
def _path(name):
    p =  os.path.join(os.getcwd(),name)
    if not os.path.exists(p):
        os.mkdir(p)
    return p
def _fs(name, id):
    p = os.path.join(_path(name),str(id))
    return int(os.path.exists(p) and os.path.getsize(p))   
def _BytesIO(name, id):
    return os.path.join(_path(name),str(id))
def _fopen(p, m):
    return open(p, m)
def _listdir(name):
    p = _path(name)
    return [os.path.join(p,f) for f in sorted(os.listdir(p), key = int)]

def _task_one(id, start, end, name, pbar):
    s = _fs(name, id)
    start += s
    pbar.update(s)
    if start <= end:
        file = _BytesIO(name, id)
    else:
        file = None
    return (start,end,file)

def _task_split(size, task_size, name, pbar):
    if size <= task_size:
        return [_task_one(0,0,size-1,name, pbar)]
    t = [(id,s,s+task_size-1) for id,s in enumerate(range(0,size,task_size))]
    td = t[-1]
    t[-1] = (td[0],td[1],size-1)
    return [_task_one(id,s,e,name, pbar) for id,s,e in t]

class downloader(object):
    def __init__(self, url, block_size=1024, task_size=1048576, name='Downloading', loop=None):
        if url and len(url[0])==3:
            self.url = url
        else:
            raise ValueError('URL must include [(url, headers, proxy)]')
        self.block_size = block_size
        self.task_size = task_size
        self.name = name
        self._len = None
        if loop:
            self.loop = loop
        else:
            self.loop = asyncio.get_event_loop()
    async def len(self, session):
        if not self._len:
            async with session.get(self.url[0][0], proxy=self.url[0][2], **_ssl) as r:
                self._len = _len(r)
                if self._len == -1:
                    await _dl2f(r, _BytesIO(self.name,0), self.block_size, _pbar(f'Unable to resume {self.name}'), 'wb')
        return self._len
    async def _next(self, s, u):
        while self._task:
            t = self._task.pop()
            await _dl(u[0],s,u[2],t[0],t[1],t[2],self.block_size,self._pbar)
    async def fetch(self):
        se = [ClientSession(headers=h) for u,h,p in self.url]
        size = await self.len(se[0])
        if size <= 0:
            await asyncio.wait([s.close() for s in se])
            return size
        self._pbar = tqdm(total=size,initial=0,unit='B',unit_scale=True, desc=self.name, ncols=ncols)
        self._task = _task_split(size, self.task_size, self.name, self._pbar)
        await asyncio.wait([self._next(s,u) for s,u in zip(se, self.url)])
        self._pbar.close()
        await asyncio.wait([s.close() for s in se])
        return size
    def start(self): 
        self.loop.run_until_complete(self.fetch())
    def close(self):
        self.loop.close()
    def save(self, file=None, Clear=True):
        if not file:
            file = open(_path(self.name)+'.data','wb')
        for p in tqdm(_listdir(self.name), ncols=ncols, desc=f'save {self.name}'):
            with open(p,'rb') as f:
                file.write(f.read())
        file.close()
        if Clear:
            self.clear()
    def clear(self):
        for p in tqdm(_listdir(self.name), ncols=ncols, desc=f'clear {self.name}'):
            os.remove(p)


if __name__ == "__main__":
    u1 = r'https://保密/webdav/保密/2020-05-27-raspios-buster-full-armhf.zip?downloadStartSecret=保密'
    headers1 = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
    "Authorization": "",
    "Connection": "keep-alive",
    "Cookie": "",
    "Host": "limour.top",
    "Sec-Fetch-Dest": "document",
    "Sec-Fetch-Mode": "navigate",
    "Sec-Fetch-Site": "same-origin",
    "Sec-Fetch-User": "?1",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36"
    }
    u2=r'http://downloads.raspberrypi.org/raspios_full_armhf/images/raspios_full_armhf-2020-05-28/2020-05-27-raspios-buster-full-armhf.zip'
    headers2 = {
    "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
    }
    a = downloader([(u1, headers1, None),[u2, headers2, None]])#同一个url重复则可以起到模拟多线程下载的效果,重复2次为2个连接
    a.start()
```

等待下载完毕后使用a.save()保存

[https://limour.lanzous.com/b015hqohg](https://limour.lanzous.com/b015hqohg) 密码:9xd4