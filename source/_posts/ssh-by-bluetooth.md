---
title: ssh by Bluetooth
tags:
  - bluetooth
  - raspberrypi
  - ssh
id: '134'
categories:
  - - Raspberry Pi
date: 2020-05-31 23:26:35
---

```
sudo apt-get install bridge-utils bluez python-dbus python-gobject
sudo ufw disable
sudo service bluetooth start
sudo update-rc.d bluetooth enable
sudo apt-get install bluez-tools
sudo nano /etc/systemd/network/pan0.netdev
输入:
[NetDev]
Name=pan0
Kind=bridge

sudo nano /etc/systemd/network/pan0.network
输入:
[Match]
Name=pan0

忽略:
#[Network]
#Address=192.168.6.6/32
#Gateway=192.168.6.6
#DHCPServer=yes
#IPForward=yes

[Network]
Address=fe80::aa16/64
DHCPServer=ipv6

sudo nano /etc/systemd/system/bt-agent.service
输入:
[Unit]
Description=Bluetooth Auth Agent

[Service]
ExecStart=/usr/bin/bt-agent -c NoInputNoOutput
Type=simple

[Install]
WantedBy=multi-user.target

sudo nano /etc/systemd/system/bt-network.service
输入:
[Unit]
Description=Bluetooth NEP PAN
After=pan0.network

[Service]
ExecStart=/usr/bin/bt-network -s nap pan0
Type=simple

[Install]
WantedBy=multi-user.target

sudo systemctl enable systemd-networkd
sudo systemctl enable bt-agent
sudo systemctl enable bt-network
sudo systemctl start systemd-networkd
sudo systemctl start bt-agent
sudo systemctl start bt-network

```

```
#!/bin/bash

IPTABLES='/usr/sbin/iptables'

# Set interface values

EXTIF='wlan0'

INTIF1='pan0'

# enable ip forwarding in the kernel

/bin/echo 1 > /proc/sys/net/ipv4/ip_forward

# flush rules and delete chains

$IPTABLES -F

$IPTABLES -X

# enable masquerading to allow LAN internet access

$IPTABLES -t nat -A POSTROUTING -o $EXTIF -j MASQUERADE

# forward LAN traffic from $INTIF1 to Internet interface $EXTIF

$IPTABLES -A FORWARD -i $INTIF1 -o $EXTIF -m state --state NEW,ESTABLISHED -j ACCEPT

#echo -e " - Allowing access to the SSH server"

$IPTABLES -A INPUT --protocol tcp --dport 22 -j ACCEPT

#echo -e " - Allowing access to the HTTP server"

```