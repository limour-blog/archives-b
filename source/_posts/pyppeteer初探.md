---
title: pyppeteer初探
tags: []
id: '1976'
categories:
  - - uncategorized
date: 2022-08-14 13:36:24
---

前面暂时解决了[倒霉微信的发信问题](https://occdn.limour.top/2242.html)，现在可以使用crontab定时每天早上将下面这个腾讯在线文档的表格的二维码发到群里了，下面还需要每天下午自动获取当天的填写情况，发送一条提醒消息到群里。这些在线表格就是不给个人提供api，腾讯文档明说要执照，金山文档就更过分了，可以跳过执照注册试用账号，结果折腾半天调通了api，发现试用账号没有获取文档的权限，我可去你\*\*的。没办法，总不能博主每天在群里吼吧，只好先来学习一下pyppeteer了。

![](https://img-cdn.limour.top/2022/08/14/62f8ed229013a.png)

用了保护所选范围+冻结+条件格式+数据验证构建的简易核酸记录表，要求在校同学每日填报

*   准备一个内存充足的服务器，先用[之间的虚拟机](https://occdn.limour.top/2244.html)折腾一下
*   sudo apt install python3-pip
*   pip3 install pyppeteer -i https://pypi.tuna.tsinghua.edu.cn/simple
*   /home/limour/.local/bin/pyppeteer-install
*   \# Chromium extracted to: /home/limour/.local/share/pyppeteer/local-chromium/588429
*   sudo apt-get install gconf-service libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxss1 libxtst6 libappindicator1 libnss3 libasound2 libatk1.0-0 libc6 ca-certificates fonts-liberation lsb-release xdg-utils wget #[解决依赖问题](https://frederick-s.github.io/2020/04/11/puppeteer-error-loading-libx11-xcb.so.1-on-ubuntu/)
*   sudo apt-get install ttf-wqy-microhei ttf-wqy-zenhei xfonts-wqy #[解决中文乱码](https://blog.csdn.net/bluecom24/article/details/39994519)

## 装个Jupyter方便调试

*   pip3 install jupyter -i https://pypi.tuna.tsinghua.edu.cn/simple
*   pip3 install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com jupyter\_contrib\_nbextensions
*   /home/limour/.local/bin/jupyter contrib nbextension install --user
*   pip3 install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com jupyter\_nbextensions\_configurator
*   /home/limour/.local/bin/jupyter nbextensions\_configurator enable --user
*   pip3 install nbconvert==5.6.1 -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
*   ifconfig grep 192.168 看一下虚拟机ip
*   nano jupyter.sh & chmod +x jupyter.sh
*   ./jupyter.sh
*   然后复制链接到本机打开就行了

```bash
#!/usr/bin/bash
/home/limour/.local/bin/jupyter notebook --no-browser --ip=`ifconfig grep 192.168 awk -F' ' '{print $2}'`
```

## 给博客首页截个图

```python
import asyncio, time
from pyppeteer import launch
async def main():
    browser = await launch(ignoreHTTPSErrors=True ,headless=True, dumpio=True, autoClose=False,
                           args=['--no-sandbox', '--window-size=1920,1080', '--disable-infobars',
                                '--disable-extensions', '--disable-gpu', '--disable-software-rasterizer',
                                '--ignore-certificate-errors', '--allow-running-insecure-content',
                                'blink-settings=imagesEnabled=false'])   # 进入无头模式
    page = await browser.newPage()           # 打开新的标签页
    await page.goto('https://limour.top/') # 访问主页
    # evaluate()是执行js的方法，js逆向时如果需要在浏览器环境下执行js代码的话可以利用这个方法
    # js为设置webdriver的值，防止网站检测
    await page.evaluate('''() =>{ Object.defineProperties(navigator,{ webdriver:{ get: () => false } }) }''')
    await page.screenshot({'path': 'limour.png'})

#     page_text = await page.content()   # 获取网页源码
#     print(page_text)
    time.sleep(1)
# asyncio.get_event_loop().run_until_complete(main()) 原生的调用方法
await main()#Jupyter调用方式，非Jupyter则用上面的调用方式
```