---
title: 快速修改HOSTS文件
tags:
  - DNS
  - hosts
  - powershell
  - Run as administrator
id: '146'
categories:
  - - 有趣技能
date: 2020-06-12 13:41:09
---

```
start notepad++ C:\WINDOWS\system32\drivers\etc\hosts -Verb RunAs
#运行 regedit.exe 修改 HKEY_CLASSES_ROOT\Microsoft.PowerShellScript.1\Shell\ 的默认为0即可
```

保存为 `Edit-Hosts.ps1` 放到你常用的地方,双击即可使用  
其中Notepad++ 可改为常用的其他文本编辑器