---
title: Win10的效率工具
tags:
  - Win10
  - 效率
id: '113'
categories:
  - - 软件分享
date: 2020-04-27 13:23:46
---

Quicker:[https://getquicker.net/](https://getquicker.net/)

Listary:[h](https://www.listary.com/)[ttps://www.listary.com/](https://www.listary.com/)

Clover:[https://clover.en.softonic.com/](https://clover.en.softonic.com/)

AutoHotkey:[https://www.autohotkey.com/](https://www.autohotkey.com/)  
`;replace CapsLock to Middle mouse button; CapsLock = Alt CapsLock`  
`$CapsLock::MButton`  
`LAlt & Capslock::SetCapsLockState, % GetKeyState("CapsLock", "T") ? "Off" : "On"`

QiuckLook:[https://github.com/QL-Win/QuickLook/releases](https://github.com/QL-Win/QuickLook/releases)  
[https://pooi.moe/QuickLook/](https://pooi.moe/QuickLook/)  
plugins:[](https://github.com/QL-Win/QuickLook/wiki/Available-Plugins)[https://github.com/QL-Win/QuickLook/wiki/Available-Plugins](https://github.com/QL-Win/QuickLook/wiki/Available-Plugins)

Geogebra:[https://www.geogebra.org/](https://www.geogebra.org/)

Zotero:[https://www.zotero.org/](https://www.zotero.org/)  
设置webdav连接到坚果云:[http://help.jianguoyun.com/?p=3168](http://help.jianguoyun.com/?p=3168)  
Zotero文献预览:[https://www.jianshu.com/p/71042f2fdd78](https://www.jianshu.com/p/71042f2fdd78)  
Rss订阅:[https://zhuanlan.zhihu.com/p/113487194](https://zhuanlan.zhihu.com/p/113487194)

* * *

Win10添加开机自启的方法

不需要管理权限的:`Win+R`输入`shell:startup`将快捷方式添加到打开的文件夹中

需要管理权限的:  
第一步,将上帝模式添加到Qiucker:[https://getquicker.net/sharedaction?code=ce9b7e99-1f36-42bc-725e-08d6600c7926](https://getquicker.net/sharedaction?code=ce9b7e99-1f36-42bc-725e-08d6600c7926)  
第二步:在打开的所有任务控制面板中搜索`计划任务`  
第三步,在计划任务的`任务计划程序库`中`创建任务`,  
`触发器`为`特定用户登录`,`操作`为`启动程序`  
第四步,在`安全选项`中`更改用户或组`,输入`Administrator`,点`检查名称`  
勾上`使用最高权限运行`,确定保存  
第五步,注销再登录,成功运行,说明没问题  
注意:启动程序一个任务只能一个程序,可以通过建新任务来指定另一个程序