---
title: 避免微软new bing标记IP
tags: []
id: '2171'
categories:
  - - uncategorized
date: 2023-03-19 07:12:25
---

看到消息：“微软直接标记ip了 梯子ip送中直接重定向到cn，测试了不是账号而是ip，就算不登录只要ip送中就一直重定向。跟有图比学真离谱啊，有图比至少还让你用”。

赶紧配置一下，避免自己的ip被标记。

## 服务器sing-box服务端规则

```json
{
  "route": {
    "rules": [
      {
        "domain_suffix": [".onion"],
        "outbound": "tor"
      },
      {
        "domain_suffix": ["openai.com"],
        "outbound": "wireguard"
      },
      {
        "domain_suffix": [".cn"],
        "outbound": "wireguard"
      },
      {
        "domain_suffix": ["check.torproject.org"],
        "outbound": "tor"
      },
      {
        "domain_suffix": ["myip.ipip.net"],
        "outbound": "wireguard"
      },
      {
        "geoip": ["cn"],
        "outbound": "wireguard"
      },
      {
        "geosite": ["cn"],
        "outbound": "wireguard"
      },
      {
        "domain_keyword": ["bing","microsoft"],
        "outbound": "wireguard"
      }
    ]
  }
}
```

## 浏览器Header Editor拓展规则

```json
{
"request": [
{
"enable": true,
"name": "bing-cn-to-www",
"ruleType": "redirect",
"matchType": "prefix",
"pattern": "https://cn.bing.com",
"exclude": "",
"group": "Bing",
"isFunction": false,
"action": "redirect",
"to": "https://www.bing.com"
}
],
"sendHeader": [
{
"enable": true,
"name": "bing",
"ruleType": "modifySendHeader",
"matchType": "all",
"pattern": "",
"exclude": "",
"group": "Bing",
"isFunction": false,
"action": {
"name": "x-forwarded-for",
"value": "51.20.xxx.xxx"
}
},
{
"enable": true,
"name": "bing2",
"ruleType": "modifySendHeader",
"matchType": "all",
"pattern": "",
"exclude": "",
"group": "Bing",
"isFunction": false,
"action": {
"name": "x-real-ip",
"value": "51.20.xxx.xxx"
}
}
],
"receiveHeader": [],
"receiveBody": []
}
```

*   Header Editor拓展[Edge商店地址](https://microsoftedge.microsoft.com/addons/detail/header-editor/afopnekiinpekooejpchnkgfffaeceko)、[Chrome商店地址](https://chrome.google.com/webstore/detail/header-editor/eningockdidmgiojffjmkdblpjocbhgh?hl=zh-CN)
*   类似的拓展：ModHeader拓展[Chrome商店地址](https://chrome.google.com/webstore/detail/modheader-modify-http-hea/idgpnmonknjnojddfkpgkljpfnnfcklj?hl=zh-CN)、[Edge商店地址](https://microsoftedge.microsoft.com/addons/detail/modheader-modify-http-h/opgbiafapkbbnbnjcdomjaghbckfkglc?hl=zh-CN)