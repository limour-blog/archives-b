---
title: 免费CDN：CFT改用CNAME
tags: []
id: '2059'
categories:
  - - uncategorized
date: 2022-10-19 11:05:03
---

[免费CDN：博客套用CFT尝试](https://occdn.limour.top/2187.html)中成功给博客套用了CFT，但是AWS分配的域名不好看，这里用CNAME将其改成自己的域名。

先在[AWS Certificate Manager](https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/welcome)中申请别名的证书。

![](https://img-cdn.limour.top/2022/10/19/634fd70bbc65d.png)

[CloudFront](https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-west-1#/)中添加别名和证书

![](https://img-cdn.limour.top/2022/10/19/634fd7c13e005.png)

DNS解析中，将别名用CNAME指向xxx.cloudfront.net，并在NPManger中添加别名反代。使用https访问别名，查看博客证书是否正确。