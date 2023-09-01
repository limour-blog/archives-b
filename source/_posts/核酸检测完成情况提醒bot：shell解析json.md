---
title: 核酸检测完成情况提醒bot：shell解析json
tags: []
id: '2055'
categories:
  - - uncategorized
date: 2022-10-08 07:07:11
---

之前做了个[核酸检测完成情况提醒bot](https://occdn.limour.top/2252.html)，来给班群三天两次进行提醒。运行了一段时间后，发现微信每15天就会踢人下线，有点恶心。因此代码里的`toUserName`就不能再写死了，得通过api每天获取。这样就被踢了就只要重新登录一下，不用再改`toUserName`了。

## 获取UserName

*   sudo apt install jq
*   nano 04.sh
*   chmod +x 04.sh

```bash
#!/bin/bash
r=`/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"name": "临八一班男生群", "token":"123456"}' \
https://limour.top/api/search_chatrooms`
r=`echo $r  jq -r '.UserName'`
echo $r > /root/task/UserName01
 
r=`/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"name": "19临八2班通知群", "token":"123456"}' \
https://limour.top/api/search_chatrooms`
r=`echo $r  jq -r '.UserName'`
echo $r > /root/task/UserName03
```

## 修改UserName获取方式

```bash
#!/bin/bash
r=`cat /root/task/UserName01`
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"fileDir": "/root/itchat/COVID.19.testing.png", "toUserName":"'$r'", "token":"123456"}' \
https://limour.top/api/send_image
```

```bash
#!/bin/bash
r=`cat /root/task/UserName03`
/usr/bin/curl -X POST \
-H "Content-Type: application/json" \
-d '{"msg": "大家记得今日填写平安复旦\n网页：https://zlapp.fudan.edu.cn/site/ncov/fudanDaily\n小程序：#小程序://复旦eHall/iOrJWtnyhqp2sos", "toUserName":"'$r'", "token":"123456"}' \
https://limour.top/api/send
```