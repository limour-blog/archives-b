---
title: 树莓派开启wifi热点
tags:
  - AP
  - raspberrypi
  - wifi
id: '499'
categories:
  - - Raspberry Pi
date: 2021-01-08 00:05:44
---

```
sudo ifconfig wlan0 down

sudo git clone https://github.com/oblique/create_ap
cd create_ap
sudo make install
sudo apt-get install util-linux procps hostapd iproute2 iw haveged dnsmasq
#安装create_ap
```

```
sudo create_ap wlan0 eth0 wifi名 密码
#测试可行后 ctrl + c 退出
```

```
cd ~
nano AutoStartAP.sh

cd /home/pi
sudo nohup sudo create_ap wlan0 eth0 wifi名 密码 >ap.log 2>&1 &

#AutoStartAP.sh 写入以上内容

sudo chmod +x AutoStartAP.sh
```

```
sudo nano /etc/rc.local

#在结尾exit 0 前添加以下内容，重启测试
su pi -c 'exec /home/pi/AutoStartAP.sh'
exit 0
```

```
sudo nano -K /etc/create_ap.conf

SHARE_METHOD=nat
IEEE80211N=1
IEEE80211AC=1

sudo systemctl start create_ap
sudo systemctl enable create_ap
```

## 更新

*   自动开启脚本

```shell
cd /home/pi
sudo killall create_ap
sudo nohup sudo create_ap --no-virt wlan0 eth0 wifiID wifiPassword >ap.log 2>&1 &
```

*   设置每天定时执行

```shell
crontab -e

20 3 * * * /home/pi/AutoStartAP.sh

crontab -l
```