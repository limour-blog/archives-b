---
title: NextCloud开启OnlyOffice
tags:
  - nextcloud
  - onlyoffice
id: '476'
categories:
  - - 建站探索
date: 2020-12-12 16:55:49
---

```
sudo -u apache php occ app:install documentserver_community
sudo -u apache php occ app:install onlyoffice
```

[![](https://img.limour.top/archives_2023/blog_wp/2020/12/image-2.webp)](https://img.limour.top/archives_2023/blog_wp/2020/12/image-2.webp)

效果展示（记得设置里关闭SSL证书验证，docker折腾到死也没弄好。。）