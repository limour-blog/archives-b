---
title: HTTP/3的QUIC连接
tags: []
id: '2062'
categories:
  - - uncategorized
date: 2022-10-20 10:55:53
---

```json
{
    "relay": {
        "server": "114.514.com",
"ip": "114.514.114.514",
        "port": 11451,
        "token": "qqrtqr",
        "udp_relay_mode": "quic",
        "congestion_controller": "bbr",
        "alpn": ["h3"],
        "disable_sni": false,
        "reduce_rtt": true
    },
    "local": {
        "port": 12080,
        "ip": "127.0.0.1"
    },
    "log_level": "info"
}
```

```powershell
# set-executionpolicy remotesigned
# New-PSDrive HKCR Registry HKEY_CLASSES_ROOT
# Set-ItemProperty HKCR:\\Microsoft.PowerShellScript.1\\Shell '(Default)' 0
.\quic-client.exe -c .\114514_quic.json
```