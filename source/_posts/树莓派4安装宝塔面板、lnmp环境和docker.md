---
title: 树莓派4安装宝塔面板、LNMP环境和docker
tags:
  - Bt-Panel
  - Docker
  - raspberrypi
id: '453'
categories:
  - - Raspberry Pi
date: 2020-12-11 22:57:48
---

```
#换源
sudo nano /etc/apt/sources.list
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi

sudo nano /etc/apt/sources.list.d/raspi.list
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui

sudo apt-get update&& sudo apt-get upgrade

```

```
#安装宝塔面板
wget -O install.sh http://download.bt.cn/install/install-ubuntu.sh && sudo bash install.sh
#之后记得在面板里放行相关端口
```

```
#到宝塔文件里编辑frpc.ini，添加内网穿透
[web01]
type = http
local_ip = 127.0.0.1
local_port = 8888
use_compression = true
subdomain = bt4
```

```
#安装docker，参见http://get.daocloud.io/#install-docker
sudo curl -sSL https://get.daocloud.io/docker  sh
```

```
#换源，如出现树莓派重启后dokcer启动失败
#处理参见：https://blog.csdn.net/frdevolcqzyxynjds/article/details/106516390
#主要原因是daemon.json必须要有Tab缩进，而不能有空格
sudo tee /etc/docker/daemon.json <<-'EOF'

{

"registry-mirrors": ["http://f1361db2.m.daocloud.io"]

}

EOF

sudo systemctl daemon-reload

```

```
#安装docker的web管理界面，参见https://www.cnblogs.com/mq0036/p/11129332.html
sudo docker pull portainer/portainer
sudo docker volume create portainer_data
sudo docker run -d -p 3375:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

```
#安装v2raya
curl -O https://jscdn.limour.top/gh/v2rayA/v2rayA@master/install/go.sh
sudo bash go.sh
wget -qO - https://raw.fastgit.org/v2rayA/v2raya-apt/master/key/public-key.asc  sudo apt-key add -
echo "deb https://raw.fastgit.org/v2rayA/v2raya-apt/master/ v2raya main"  sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
sudo apt install v2raya
sudo systemctl enable v2ray && sudo systemctl start v2ray
#登录127.0.0.1:2017,修改sock5端口为1080
```

```
#安装proxychains4
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng/
./configure --prefix=/usr --sysconfdir=/etc
sudo make && sudo make install
sudo make install-config
sudo nano /etc/proxychains.conf
#注释掉默认的这个配置
#socks4         127.0.0.1 9050
#添加下面这个配置
socks5  127.0.0.1 1080
proxychains4 curl www.google.com
```