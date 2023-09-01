---
title: Aria2下载百度网盘的配置文件
tags:
  - aria2c
  - baidupcs
id: '506'
categories:
  - - Raspberry Pi
date: 2021-01-18 16:11:40
---

```
disk-cache=32M
continue=true
max-connection-per-server=3
min-split-size=8M
peer-id-prefix=-TR2770-
bt-seed-unverified=true
bt-save-metadata=true
enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
check-certificate=false
follow-torrent=true
enable-peer-exchange=true
seed-ratio=1.0
seed-time=1440
max-overall-upload-limit=30000
max-upload-limit=10000
disable-ipv6=true
enable-dht=true
bt-enable-lpd=true
enable-peer-exchange=true
rpc-listen-port=57663
listen-port=57665
dht-listen-port=57666
bt-detach-seed-only=true
dir=/mnt/u/aria2
dht-file-path=/mnt/a/dht.dat
bt-tracker=udp://93.158.213.92:1337/announce,http://93.158.213.92:1337/announce,udp://62.210.97.59:1337/announce,udp://151.80.120.112:2710/announce,udp://151.80.120.114:2710/announce,http://62.210.97.59:1337/announce,udp://208.83.20.20:6969/announce,udp://185.181.60.67:80/announce,udp://194.182.165.153:6969/announce,udp://5.206.3.65:6969/announce,udp://37.235.174.46:2710/announce,udp://89.234.156.205:451/announce,udp://92.223.105.178:6969/announce,udp://176.113.71.60:6961/announce,http://176.113.71.60:6961/announce,udp://207.241.231.226:6969/announce,udp://207.241.226.111:6969/announce,udp://51.15.40.114:80/announce,udp://184.105.151.164:6969/announce,http://184.105.151.164:6969/announce,udp://212.47.227.58:6969/announce,udp://31.210.170.169:2710/announce,udp://46.148.18.250:2710/announce,udp://51.15.55.204:1337/announce,udp://51.158.23.91:6969/announce,udp://217.76.183.53:80/announce,udp://185.83.214.123:6969/announce,udp://46.148.18.254:2710/announce,http://95.107.48.115:80/announce,http://51.15.55.204:1337/announce,http://185.83.214.123:6969/announce,udp://78.46.225.225:2710/announce,udp://207.246.121.172:2000/announce,udp://5.226.148.20:6969/announce,http://78.46.225.225:2710/announce,udp://94.237.82.46:6969/announce,udp://91.149.192.31:6969/announce,udp://94.224.67.24:6969/announce,udp://103.196.36.31:6969/announce,udp://62.171.179.41:80/announce,udp://182.150.53.61:8080/announce,udp://212.1.226.176:2710/announce,udp://173.82.240.104:6969/announce,udp://210.210.158.134:8866/announce,http://182.150.53.61:8080/announce,http://210.210.158.134:8866/announce,http://191.19.224.81:6969/announce,udp://51.254.249.186:6969/announce,udp://220.173.39.56:6969/announce,udp://52.58.128.163:6969/announce,udp://111.235.252.90:6969/announce,udp://218.5.42.68:2710/announce,http://51.254.249.186:6969/announce,http://101.200.58.197:6969/announce,http://220.173.39.56:6969/announce,http://122.116.144.108:6969/announce,http://78.30.254.12:2710/announce,http://54.37.106.164:80/announce,http://91.207.136.85:80/announce,udp://176.113.68.67:6961/announce,udp://51.15.3.74:6969/announce,udp://139.99.100.97:8080/announce,http://176.113.68.67:6961/announce,http://149.28.95.5:2710/announce,http://51.255.140.206:80/announce,http://54.39.179.91:6699/announce
user-agent=netdisk;11.5.3;YAL-AL00;android-android;10;JSbridge4.4.0;jointBridge;1.1.0;
rpc-secret=
header=Cookie: BDUSS=; STOKEN=;
```

自行设置密码rpc-secret=

Chrome通过F12查看得到BDUSS和STOKEN填入上述文件最后的header项中

之后通过请参照[https://github.com/syhyz1990/baiduyun](https://github.com/syhyz1990/baiduyun)