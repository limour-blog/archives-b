---
title: 树莓派通过docker编译R和安装RStudio
tags:
  - Docker
  - R
  - Rstudio
id: '583'
categories:
  - - Raspberry Pi
date: 2021-03-14 22:02:17
---

```
docker run --interactive --tty --rm --name rserver --publish 18787:8787 --volume /mnt/data:/home/rstudio --workdir /root arturklauser/raspberrypi-rstudio-build-env:1.2.5019-buster-buildx /bin/bash

nano /etc/profile
export JAVA_HOME=/home/rstudio/os_bin/jdk1.8.0_281/
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
source /etc/profile

cd /home/rstudio/R/proxychains-ng/
make install
make install-config
sudo nano -K /etc/proxychains.conf
http 172.17.0.1 10808

proxychains4 apt install gfortran

cd /home/rstudio/os_bin/pcre2-10.36
make install

cd /home/rstudio/os_bin/curl-7.75.0 (./configure --with-ssl)
make install

cd /home/rstudio/os_bin/R-4.0.4
./configure --enable-R-shlib --with-x=no
make
make install

docker commit -a "limour.top" -m "R-4.0.4" -p 94d5e8abebf7 rserver:v404

proxychains4 docker pull arturklauser/raspberrypi-rstudio-server-deb:1.2.5019-buster-buildx

docker image save arturklauser/raspberrypi-rstudio-server-deb  tar xO --wildcards '*/layer.tar'  tar x

docker run --interactive --tty --rm --name rserver --publish 18787:8787 --volume /mnt/data:/home/rstudio --workdir /root rserver:v404 /bin/bash

cd /home/rstudio/R/rstudio/

proxychains4 apt-get update

proxychains4 apt install ./rstudio-server-1.2.5019-1~r2r_armhf.deb

sudo groupadd rstudio
sudo useradd rstudio -g rstudio
passwd rstudio

nano -K /etc/rstudio/rserver.conf
rsession-which-r=/usr/local/bin/R
rstudio-server restart

docker commit -a "limour.top" -m "R-4.0.4" -p 0e8745e3f09e rserver:rt404

```