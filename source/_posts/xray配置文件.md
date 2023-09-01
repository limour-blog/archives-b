---
title: Xray配置文件
tags: []
id: '641'
categories:
  - - 计算机相关
date: 2021-09-19 20:09:06
---

*   下载并解压核心 [`Xray-core`](https://github.com/XTLS/Xray-core/releases)
*   配置文件示例，`xui2.json`：

```
{
  "log": {
  "loglevel": "warning" 
  },
  "inbounds": [
    {
      "port": "20808",
      "protocol": "socks",
      "settings": {
        "auth": "noauth",
        "udp": true
      }
    },
    {
      "port": "20809",
      "protocol": "http",
      "settings": {}
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "xxx.top",
            "port": 443,
            "users": [
              {
                "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", //填写你的 UUID
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "grpc",
        "security": "tls",
        "grpcSettings": {
          "serviceName": "xxx.top" //填写你的 ServiceName
        }
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
"rules": [
// 3.1 广告域名屏蔽
{
"type": "field",
"domain": [
"geosite:category-ads-all"
],
"outboundTag": "block"
},
// 3.2 国内域名直连
{
"type": "field",
"domain": [
"geosite:cn"
],
"outboundTag": "direct"
},
// 3.3 国内IP直连
{
"type": "field",
"ip": [
"geoip:cn",
"geoip:private"
],
"outboundTag": "direct"
}
    ]
  }
}
```

*   `proxy.ps1` 文件示例：

```
# set-executionpolicy remotesigned
# New-PSDrive HKCR Registry HKEY_CLASSES_ROOT
# Set-ItemProperty HKCR:\\Microsoft.PowerShellScript.1\\Shell '(Default)' 0
.\xray.exe run -c .\xui2.json
```