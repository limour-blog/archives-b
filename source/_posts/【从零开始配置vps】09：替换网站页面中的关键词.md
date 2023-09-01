---
title: 【从零开始配置VPS】09：替换网站页面中的关键词
tags: []
id: '2120'
categories:
  - - uncategorized
date: 2023-01-30 09:11:27
---

```apache2
<IfModule mod_substitute.c>
  <Location />
    AddOutputFilterByType SUBSTITUTE text/html
    Substitute s/cdn.jsdelivr.net/jscdn.limour.top/ni
  </Location>
</IfModule>
```

*   cd ~/app/WordPress

*   nano jscdn.conf

```bash
sudo docker cp ./jscdn.conf \
 wordpress-wordpress-1:/etc/apache2/conf-enabled/jscdn.conf

sudo docker exec -it wordpress-wordpress-1 \
cp /etc/apache2/mods-available/substitute.load \
/etc/apache2/mods-enabled/substitute.load 

```