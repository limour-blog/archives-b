---
title: 核酸检测完成情况提醒bot开发记录
tags: []
id: '1980'
categories:
  - - uncategorized
date: 2022-08-15 15:40:01
---

## 需求

*   每天早上核酸检测即将开始，定时发送在线表格二维码到班级微信群
*   每天下午核酸检测即将结束，定时检测未完成填写的同学名单，发送提醒信息到班级微信群
*   在线表格设计如下

![](https://img-cdn.limour.top/2022/08/14/62f8ed229013a.png)

用了保护所选范围+冻结+条件格式+数据验证构建的简易核酸记录表，要求在校同学每日填报

## 功能拆分

*   低功耗小主机，如树莓派，运行微信，开放发信API到公网，[demo点此](https://occdn.limour.top/2242.html)
*   爬虫采集腾讯文档在线表格内容，[demo点此](https://occdn.limour.top/2249.html)
*   通过爬虫采集的内容，分析未完成填写的同学名单，通过发信API发送提醒信息到班级微信群
*   最终效果见下图

![](https://img-cdn.limour.top/2022/08/15/62fa52398a154.png)

左图是早上定时提醒的效果，右图分别是有同学未记录和全部已记录的效果

## 发信API

```python
#!/usr/bin/python3
import itchat
itchat.auto_login(True, enableCmdQR=2)
 
import bottle
 
def verifyToken(func):
    def wrapper(*args, **kwargs):
        token = bottle.request.json.get('token')
        if token != 'a123456':
            return dict(BaseResponse='invalid token!')
        else:
            return func(*args, **kwargs)
    return wrapper
    
@bottle.route('/api/send_image', method='POST')
@verifyToken
def api_send_image():
    fileDir = bottle.request.json.get('fileDir')
    toUserName = bottle.request.json.get('toUserName')
    return dict(itchat.send_image(fileDir, toUserName))
    
@bottle.route('/api/send', method='POST')
@verifyToken
def api_send():
    msg = bottle.request.json.get('msg')
    toUserName = bottle.request.json.get('toUserName')
    return dict(itchat.send(msg, toUserName))
    
@bottle.route('/api/search_friends', method='POST')
@verifyToken
def api_search_friends():
    remarkName = bottle.request.json.get('remarkName')
    usr = itchat.search_friends(remarkName=remarkName)
    if not usr:
        return dict(BaseResponse='failed')
    usr = usr[0]
    return dict(BaseResponse='succeed', UserName=usr.userName)
    
@bottle.route('/api/search_chatrooms', method='POST')
@verifyToken
def api_search_chatrooms():
    name = bottle.request.json.get('name')
    group =  itchat.search_chatrooms(name=name)
    if not group:
        return dict(BaseResponse='failed')
    group = group[0]
    return dict(BaseResponse='succeed', UserName=group.userName)
 
bottle.run(host='localhost', port=2001, debug=True)
```

*   依赖见[demo点此](https://occdn.limour.top/2242.html)
*   screen /home/pi/itchat\_api.py
*   下面是测试，通过后修改token，通过反代将服务暴露到公网，并加上https

```bash
#!/bin/bash
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"fileDir": "./itchat/COVID.19.testing.png", "toUserName":"filehelper", "token":"a123456"}' \
http://localhost:2001/api/send_image
 
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"msg": "./itchat/COVID.19.testing.png", "toUserName":"filehelper", "token":"a123456"}' \
http://localhost:2001/api/send
 
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"remarkName": "没有这个备注", "token":"a123456"}' \
http://localhost:2001/api/search_friends
 
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"remarkName": "Limour", "token":"a123456"}' \
http://localhost:2001/api/search_friends
 
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"name": "没有这个群", "token":"a123456"}' \
http://localhost:2001/api/search_chatrooms
 
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"name": "相亲相爱一家人", "token":"a123456"}' \
http://localhost:2001/api/search_chatrooms
```

## 早上提醒

```python
#!/bin/bash
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"fileDir": "./itchat/COVID.19.testing.png", "toUserName":"filehelper", "token":"a123456"}' \
http://localhost:2001/api/send_image
```

*   toUserName从filehelper改为班级群的group.userName
*   crontab -e
*   10 7 \* \* \* /home/pi/task/01.sh
*   获取班级群的group.userName的方法如下

```bash
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"name": "临八一班男生群", "token":"a123456"}' \
http://localhost:2001/api/search_chatrooms
```

## 下午提醒

### 通过puppeteer获取腾讯文档在线表格内容

```js
#!/usr/bin/env node
const puppeteer = require('puppeteer-extra')
// add stealth plugin and use defaults (all evasion techniques)
const StealthPlugin = require('puppeteer-extra-plugin-stealth')
puppeteer.use(StealthPlugin())
const fs = require('fs')
 
puppeteer.launch({ headless: true }).then(async browser => {
    const context = browser.defaultBrowserContext();
    context.overridePermissions('https://docs.qq.com', ['clipboard-read'])
    console.log('Running tests..')
    const page = await browser.newPage()
    await page.goto('https://docs.qq.com/sheet/id')
    console.log('Open sheet.html')
    
    await page.waitForSelector('#canvasContainer > div.excel-container > canvas')
    await page.waitForTimeout(500)
    await page.click('#canvasContainer > div.excel-container > canvas');
    await page.waitForTimeout(100)
    await page.focus('#canvasContainer > div.excel-container > canvas')
    console.log('focus sheet1')
    
    await page.waitForTimeout(100)
    
    await page.keyboard.down('Control');
    await page.waitForTimeout(30)
    await page.keyboard.press('KeyA');
    await page.waitForTimeout(10)
    await page.keyboard.up('Control');
    console.log('send Ctrl A')
    
    await page.waitForTimeout(1000)
    
    await page.keyboard.down('Control');
    await page.waitForTimeout(21)
    await page.keyboard.press('KeyC');
    await page.waitForTimeout(12)
    await page.keyboard.up('Control');
    console.log('send Ctrl C')
    
    await page.waitForTimeout(2000)
    
    const table = await page.evaluate(() => navigator.clipboard.readText())
    console.log(table)
    fs.writeFile('test.txt', table, err => {
      if (err) {
        console.error(err)
        return
      }
      //文件写入成功。
    })
    console.log('get clipboard')
    
    await page.screenshot({ path: 'testresult.png', fullPage: true })
    await browser.close()
    console.log(`All done, check the screenshot. ✨`)
    process.exit()
})
```

### 使用Python解析上一步获取的内容

```python
#!/usr/bin/env python3
from os import system, access, R_OK
import sys
system(f"rm split_table.pkl")
system(f"./wecomchan.py 'exec txtable.js'")
system(f"rm test.txt")
for i in range(3): # 尝试三次
    system(f"./txtable.js")
    if access('test.txt', R_OK):
        with open('test.txt', encoding="utf-8") as f:
            table = f.read().strip()
            print(table)
            if table:
                system(f"./wecomchan.py 'exec txtable.js succeed'")
                break
    system(f"./wecomchan.py 'exec txtable.js failed once'")
else:
    system(f"./wecomchan.py 'exec txtable.js failed'")
    sys.exit(1)
system(f"rm test.txt")
table = table.splitlines()
import datetime
today=datetime.date.today()
formatted_today=today.strftime('%-m月%-d日')
for line in table:
    if line.startswith(formatted_today):
        print(line)
        today_data = line
        break
else:
    system(f"./wecomchan.py 'txtable.py failed: line.startswith(formatted_today) is False'")
    sys.exit(1)
table = table[0:3]
table.append(today_data)
split_table = list()
for line in table:
    line = line.split('\t')
    split_table.append(line)
#     print(line)
split_table[0] = list(filter(None, split_table[0]))
for i in range(1,len(split_table)):
    split_table[i] = split_table[i][0:len(split_table[0])]
import pickle
with open("split_table.pkl", "wb") as tf:
    pickle.dump(split_table,tf)
```

### 进行下午提醒

```python
#!/usr/bin/env python3
from os import system, access, R_OK, chdir
chdir('/home/limour/02')
import sys
system(f"./txtable.py")
if not access('split_table.pkl', R_OK):
    sys.exit(1)
import pickle
with open("split_table.pkl", "rb") as tf:
    table = pickle.load(tf)
r1 = table[0][2:] #需要填写的人
r4 = table[3][2:] #已经填写的人
r5 = list(r1[x] for x in range(len(r4)) if r4[x] == '') #未填写的人
if r5:
    info = '@'+ ' @'.join(r5)
    info = f"核酸检测即将结束，请以下同学及时记录自己的检测情况(n={len(r5)})：\n{info}"
else:
    info = f"今日核酸/抗原已全部完成！"
system(f"./wecomchan.py '{info}'")
info = info.replace('\n', '\\n')
system(f'''/usr/bin/curl -X POST \\
-H "Content-Type: application/json" \\
-d '{{"msg": "{info}", "toUserName":"filehelper", "token":"a123456"}}' \\
https://limour.top/api/send''')
```

*   toUserName从filehelper改为班级群的group.userName
*   tzselect #修改时区
*   sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
*   date +"%m-%d-%H-%M-%S"#查询系统当前时间
*   crontab -e
*   0 16 \* \* \* /home/limour/02/reminder.py
*   \# [crontab 执行时间计算器](https://tool.lu/crontab/)