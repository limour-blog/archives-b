---
title: 调小MTU来获得更稳定的网络
tags: []
id: '2126'
categories:
  - - uncategorized
date: 2023-02-02 10:40:42
---

电脑连某些WIFI总是不稳定，时不时断流，干脆调小MTU来保证长连接更稳定。

## Win10

*   ping -l 1372 -f www.baidu.com
*   netsh interface ipv4 show subinterfaces
*   netsh interface ipv4 set subinterface "WLAN" mtu=1400 store=persistent

## Ubuntu

```ini
[Unit]
Description=Reset-mtu
After=network.target
[Service]
ExecStart=/usr/sbin/ifconfig ens33 mtu 1400
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

*   netstat -i
*   ping -s 1372 -M do www.baidu.com
*   sudo nano /etc/systemd/system/rmtu.service
*   sudo systemctl enable rmtu && sudo systemctl start rmtu
*   cat /sys/class/net/ens33/mtu