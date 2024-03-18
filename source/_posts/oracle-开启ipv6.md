---
title: Oracle 开启IPv6
tags: []
id: '1116'
categories:
  - - 建站探索
  - - 计算机相关
date: 2021-10-24 01:08:25
---

## 第一步 配置IPv6

[甲骨文云免费VPS系列二：Oracle甲骨文云ARM后台升降级配置流程，无限刷临时公共IP，预留IP应用，甲骨文云开启原生IPV6、后台端口规则设置流程，引导卷删除与容量设置](https://www.youtube.com/watch?v=rTYDSYjFoBk)

## 第二步 添加AAAA记录，开启Nginx监听

*   ipv6only=on只能添加一次，否则nginx会报错端口重复

```
listen [::]:80 ipv6only=on;
listen [::]:443 ssl http2;
```

*   测试1：netstat -tuln
*   测试2：[http://ipv6-test.com/validate.php](http://ipv6-test.com/validate.php)

## 第三步 获取IPv6地址

*   ifconfig
*   dhclient -6 ens3 # 上面有IPv6地址则不用再设置
*   ping6 ipv6.google.com # 注意 入站规则里ipv6要放开所有端口，否则ping不通
*   ping limour.top # 注意 入站规则里ipv4要放开所有端口，否则ping不通
*   [v6t.ipip.net](http://v6t.ipip.net/)

## 第四步 可能的问题

[Oracle Cloud 甲骨文云启用原生 IPv6 地址详细教程 - 简单、通用、免费、双栈更香](https://vircloud.net/exp/oracle-cloud-ipv6.html)

## 第五步 安装wireguard

*   uname -r # 5.6 内置了 wireguard module
*   modprobe wireguard # 启动`wireguard`模块

## 第六步 急救

*   停止实例
*   启动实例
*   创建命令 sudo ufw default allow

## 第七步 安全 ufw (没用)

*   sudo ufw enable
*   sudo ufw status
*   sudo ufw deny 54321
*   sudo ufw disable

## 第八步 安全 iptables

*   /usr/sbin/iptables -F
*   /usr/sbin/iptables -A INPUT ! -s 127.0.0.1 -p tcp --dport 54321 -j DROP
*   /usr/sbin/iptables -A INPUT ! -s 127.0.0.1 -p tcp --dport 8888 -j DROP
*   /usr/sbin/iptables -A INPUT ! -s 127.0.0.1 -p tcp --dport 5080 -j DROP
*   /usr/sbin/iptables -I INPUT -s \*\*\*.\*\*\*.\*\*\*.\*\*\* -j DROP

[Linux下iptables 禁止端口和开放端口](https://blog.csdn.net/zht666/article/details/17505789)

[![](https://img.limour.top/archives_2023/blog_wp/2021/10/8ea6b47ff0fb78f9a848e9648ec364c.jpg)](https://img.limour.top/archives_2023/blog_wp/2021/10/8ea6b47ff0fb78f9a848e9648ec364c.jpg)