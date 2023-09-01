---
title: WordPress修改端口
tags:
  - web
  - 笔记
id: '40'
categories:
  - - 建站探索
date: 2020-04-15 12:57:25
---

第一步:向/etc/httpd/conf/httpd.conf 中添加 `listen 10080`  
然后# `systemctl restart httpd`

第二步:修改主题的functions.php 向其添加  
`remove_filter('template_redirect','redirect_canonical');`

第三步:同时修改设置里的site和home为  
`site http://localhost:10080/wordpress  
home http://localhost:10080`

第四步:修改菜单和其他链接,添加端口  
例如首页菜单链接修改为`http://localhost:10080`