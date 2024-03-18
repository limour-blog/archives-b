---
title: 让NPM面板只允许CDN回源IP访问
tags: []
id: '2000'
categories:
  - - uncategorized
date: 2022-09-02 05:52:22
---

之前用[docker安装NPM面板](https://occdn.limour.top/1997.html)，并套上了CF的免费CDN，做好了规则防护。现在还差一个防止通过固定Hostname然后扫描全网ip来反查源站的保护。

![](https://img.limour.top/archives_2023/2022/09/02/6311991bb6b3f.webp)

```bash
#!/bin/bash
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -j DROP
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 173.245.48.0/20 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 103.21.244.0/22 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 103.22.200.0/22 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 103.31.4.0/22 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 141.101.64.0/18 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 108.162.192.0/18 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 190.93.240.0/20 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 188.114.96.0/20 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 197.234.240.0/22 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 198.41.128.0/17 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 162.158.0.0/15 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 104.16.0.0/13 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 104.24.0.0/14 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 172.64.0.0/13 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -p tcp -m multiport --dports 80,443 -s 131.0.72.0/22 -j RETURN
/usr/sbin/iptables -I DOCKER-USER -s 172.16.0.0/12 -j RETURN
```

*   nano docker\_iptables.sh && chmod +x docker\_iptables.sh
*   ./docker\_iptables.sh