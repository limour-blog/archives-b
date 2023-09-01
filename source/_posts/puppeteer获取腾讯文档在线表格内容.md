---
title: puppeteer获取腾讯文档在线表格内容
tags: []
id: '1978'
categories:
  - - uncategorized
date: 2022-08-15 02:43:47
---

[pyppeteer初探](https://occdn.limour.top/2246.html)完了，发现并不怎么好用，还是用原生的puppeteer吧，以下是在前面[pyppeteer初探](https://occdn.limour.top/2246.html)已安装依赖的前提下的继续。

*   sudo apt install libzmq3-dev pkg-config
*   sudo apt install nodejs npm
*   npm config set registry https://registry.npm.taobao.org
*   npm config get registry
*   sudo npm install n -g
*   sudo n stable
*   退出shell再重进
*   安装[ijavascript](https://github.com/n-riesco/ijavascript)
*   npm install puppeteer --save
*   npm install puppeteer-extra puppeteer-extra-plugin-stealth
*   sudo apt-get install -y libgbm-dev
*   nano txtable.js
*   chmod +x txtable.js
*   ./txtable2.js

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
    await page.goto('https://docs.qq.com/sheet/DVFRPa2lYZ3JtdEtn?tab=BB08J2')
    console.log('Open sheet.html')
    
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