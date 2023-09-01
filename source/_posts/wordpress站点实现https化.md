---
title: Wordpress站点实现Https化
tags:
  - Https
  - SSL
  - WordPress
  - 泛域名
id: '109'
categories:
  - - 建站探索
date: 2020-04-24 16:25:38
---

获得证书

第一步:#`yum install -y python3 && pip3 install certbot`  
安装完毕后，运行#`certbot --help`可以查看该工具的命令详情。

第二步,停止Apache：#`systemctl stop httpd`,同时开放433端口和80端口

第三步,运行#`certbot certonly --standalone -d limour.top`获取证书  
如果失败,多试几次,一次一个域名比较容易成功,成功后有Congratulations显示

证书路径一般如下,记录下来:  
`"cert": "/etc/letsencrypt/live/limour.top/fullchain.pem"`  
`"key": "/etc/letsencrypt/live/limour.top/privkey.pem"`

后续运行#`certbot certificates`命令可再次查看获取到申请的证书及所在目录  
证书有效期为三个月,到期前运行#`certbot renew --force-renew`可续期

* * *

配置SSL

第一步,进入httpd模块配置目录#`cd /etc/httpd/conf.d`  
#`ls -l`检查是否有且只有一个`ssl.conf`文件,是则直接进入下一步  
没有则可#`yum -y install mod_ssl`进行安装  
如果yum显示模块已经存在,先#`rm ssl.*`清理可能的配置文件  
通过#`yum -y reinstall mod_ssl`重新安装,直到出现且仅有一个`ssl.conf`

第二步,确保模块和配置引入:#`nano /etc/httpd/conf/httpd.conf`  
Ctrl+W 查找`IncludeOptional conf.d/*.conf`确保其存在且未被注释  
Ctrl+W 查找`Include conf.modules.d/*.conf`确保其存在且未被注释  
Ctrl+X 退出nano 如果修改了文件,退出时,输入y回车,再次回车可保存

第三步,开启ssL:#`nano ssl.conf`,确保`Listen 443 https`存在且未被注释  
修改`SSLCertificateFile`后的路径为获得证书中`"cert"`的路径  
修改`SSLCertificateKeyFile`后的路径为获得证书中`"key"`的路径  
保存退出,重启httpd #`systemctl start httpd`

* * *

配置WordPress

登录网站后台,在设置-常规中修改`WordPress地址（URL）`和`站点地址（URL）`为`https://limour.top`,如有图片,可修改背景图片链接,清空浏览器缓存,打开`https://limour.top`即可看到地址栏绿锁

* * *

其他无关设置

确保`/etc/httpd/modules`中有`mod_proxy`相关的一堆模块  
确保`/etc/httpd/conf.modules.d`中有`00-proxy.conf`  
在`/etc/httpd/conf.d`目录下#`nano zproxy.conf`  
输入`ProxyPass /jsonrpc http://127.0.0.1:你的端口/jsonrpc`项  
可转发aria2 Web控制台的连接,实现https控制aria2下载,保护token  
其他反向代理可加如下项确保重定向不跳代理,不过测试时记得清浏览器缓存  
`ProxyPassReverse /jsonrpc http://127.0.0.1:你的端口/jsonrpc`  
`ProxyPass /images !`这句表示以`/image`开始的URL不被代理  
`ProxyPassMatch ^/images !`这句也表示以`/image`开始的URL不被代理

```
<Directory "/var/www/html/AriaNg">
AllowOverride AuthConfig
</Directory>
```

在#`nano /etc/httpd/conf/httpd.conf`中添加上述项  
在/`var/www/html/AriaNg/`目录中建立`.htaccess`文件  
文件中键入以下内容:

```
AuthName "This page is private"
AuthType Basic
AuthUserFile /var/www/apasswd
require valid-user
```

同时运行#`htpasswd -c /var/www/apasswd manger`建立用户密码文件  
#`systemctl start httpd`重启httpd即可实现对AriaNg目录的登录验证  
再次提醒,测试时需要清理浏览器缓存,不然会出现奇怪的问题

#`nano /var/www/html/nextcloud/config/config.php`相关内容改为  
`'trusted_domains'=>array (0=>'47.102.216.189',1=>'limour.top',),  
'overwrite.cli.url' => 'https://47.102.216.189/nextcloud',  
'overwriteprotocol'=>'https'`可实现NextCloud的https化

```
#!/bin/bash
systemctl stop httpd
certbot renew --force-renew
systemctl start httpd
```

```
nano /etc/crontab
0 1 9 * * root /root/updatecert.sh > /root/upclog.txt
```

```
<IfModule mod_rewrite.c>
RewriteEngine on
RewriteBase /
RewriteCond %{SERVER_PORT} !^443$
RewriteRule (.*) https://%{SERVER_NAME}/$1 [R=301,L]
</IfModule>
```

```
#certbot certonly  -d "*.limour.top" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory
```

```
>nslookup -q=TXT _acme-challenge.limour.top
```

```
<VirtualHost _default_:443>
#......
SSLCertificateFile /etc/letsencrypt/live/limour.top-0001/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/limour.top-0001/privkey.pem
#......
</VirtualHost>

```