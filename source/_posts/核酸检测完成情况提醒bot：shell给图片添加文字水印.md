---
title: 核酸检测完成情况提醒bot：shell给图片添加文字水印
tags: []
id: '2077'
categories:
  - - uncategorized
date: 2022-11-07 04:38:53
---

上辈子造了孽，这辈子用微信。[核酸检测完成情况提醒bot：shell解析json](https://occdn.limour.top/2415.html)后，微信又出幺蛾子了，核酸检测表格的图片被屏蔽了，只有自己能看到，群里其他人看不到。现在需要每天给图片加点料，避开微信的检测。

## 安装依赖

*   apt update
*   apt install imagemagick

## 添加水印的脚本

```bash
#!/usr/bin/env bash
text=`date`
convert /root/itchat/COVID.19.testing_raw.png \
-gravity southeast -fill black -pointsize 16 \
-draw "text 5,5 '$text'" \
/root/itchat/COVID.19.testing.png
```

*   nano sb\_wechat.sh && chmod +x sb\_wechat.sh

## 添加定时执行

*   crontab -e
*   0 7 \* \* \* /root/itchat/sb\_wechat.sh
*   crontab -l