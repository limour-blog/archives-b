---
title: Wordpress开启防盗链
tags:
  - WordPress
  - 图片
  - 防盗链
id: '124'
categories:
  - - 建站探索
date: 2020-05-15 23:09:50
---

.htaccess 文件常见到 \[NC\]\[L\]\[R\]\[F\] 几个字符的含义  
1、NC：no case，不区分大小写，忽略大小写；  
2、R：redirect，重定向；  
3、F：forbidden，禁止访问;  
4、L：last，表示已是最后一条规则，.htaccess 文件解析将退出.

```
RewriteEngine on
RewriteBase /wp-content/uploads/
RewriteCond %{HTTP_REFERER} ^$ [NC]
RewriteCond %{HTTP_REFERER} !limour.top [NC]
RewriteCond %{HTTP_REFERER} !zhuaxia.com [NC]
RewriteCond %{HTTP_REFERER} !xianguo.com [NC]
RewriteCond %{HTTP_REFERER} !google.com [NC]
RewriteCond %{HTTP_REFERER} !feedburner.com [NC]
RewriteCond %{HTTP_REFERER} !feedsky.com [NC]
RewriteCond %{HTTP_REFERER} !baidu.com [NC]
RewriteCond %{HTTP_REFERER} !soso.com [NC]
RewriteCond %{HTTP_REFERER} !sogou.com [NC]
RewriteCond %{HTTP_REFERER} !360.cn [NC]
RewriteRule .*\.(gifjpgpngrar)$ https://i.loli.net/2020/05/15/5BnMQzU74teqm2p.png [L]
```

以后用图片选择 区块-图像-媒体库 插入

![](https://img-cdn.limour.top/blog_wp/2020/04/589c3057c63f8-1024x576.jpg)