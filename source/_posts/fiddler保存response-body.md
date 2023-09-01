---
title: Fiddler保存Response body
tags:
  - Fiddler
  - Response body
id: '149'
categories:
  - - 有趣技能
date: 2020-06-16 19:38:07
---

```
static function OnDone(oSession: Session) {
var fileName="";
var fileIndex = oSession.RequestHeaders.RequestPath.LastIndexOf ("/");
if (fileIndex > 0)
{
fileName = oSession.RequestHeaders.RequestPath.Substring (fileIndex+1);
}
if (String.IsNullOrEmpty(fileName))
{
fileName = Guid.NewGuid().ToString();
}
var saveDir="H:\\test\\";
if (!System.IO.Directory.Exists(saveDir))
{
System.IO.Directory.CreateDirectory(saveDir);
}
oSession.SaveResponseBody(saveDir + fileName);
}
```