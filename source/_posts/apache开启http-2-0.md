---
title: Apache开启Http/2.0
tags:
  - http2
id: '524'
categories:
  - - 建站探索
date: 2021-02-01 15:58:01
---

[![](https://img.limour.top/archives_2023/blog_wp/2021/02/image.webp)](https://myssl.com/limour.top?status=success#basic)

```
yum install -y expat-devel 
yum install -y epel-release
cd /etc/yum.repos.d && wget https://repo.codeit.guru/codeit.el`rpm -q --qf "%{VERSION}" $(rpm -q --whatprovides redhat-release)`.repo
yum info httpd

rm -rf /etc/httpd/
yum reinstall httpd
systemctl start httpd
systemctl status httpd -l
systemctl enable httpd


yum reinstall -y php-fpm
systemctl restart php-fpm
systemctl status php-fpm
systemctl enable php-fpm -l


cd /etc/httpd/conf.modules.d/
nano -K 02-fpm.conf
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
<FilesMatch \.php$>
SetHandler "proxy:fcgi://127.0.0.1:9000"
</FilesMatch>
cd /etc/httpd/conf/
nano -K httpd.conf
DirectoryIndex index.php index.html
systemctl restart httpd
http://limour.top/nextcloud/index.php

yum -y reinstall mod_ssl
certbot certificates
cd /etc/letsencrypt/live/
ll
cd /etc/httpd/conf.d/
nano -K ssl.conf
SSLCertificateFile /etc/letsencrypt/live/limour.top-0001/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/limour.top-0001/privkey.pem
#SSLCertificateChainFile /etc/letsencrypt/live/limour.top-0001/chain.pem
cd /etc/httpd/conf/
nano -K httpd.conf
<Directory "/var/www/html">
Header always add Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
</Directory>
systemctl restart httpd
https://limour.top/nextcloud/index.php/login



cd /etc/httpd/conf/
nano -K httpd.conf
<Directory />
Order deny,allow
</Directory>
<Directory "/var/www/html">
    AllowOverride All
</Directory>
systemctl restart httpd

cd /etc/httpd/conf.d/
nano -K zmyapp.conf
ProxyPass /aria2 http://127.0.0.1:57663/jsonrpc
<Directory "/var/www/html/AriaNg">
AllowOverride AuthConfig
</Directory>
<VirtualHost *:443>
ServerName limour.top
DocumentRoot /var/www/html
SSLEngine on
SSLProtocol +TLSv1.2 +TLSv1.3
SSLProxyProtocol +TLSv1.2 +TLSv1.3
SSLCipherSuite "EECDH+AES128:EECDH+AES256:+SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:RSA+3DES:!DSS"
SSLProxyCipherSuite "EECDH+AES128:EECDH+AES256:+SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:RSA+3DES:!DSS"
SSLCertificateFile /etc/letsencrypt/live/limour.top/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/limour.top/privkey.pem
</VirtualHost>
systemctl restart httpd

cd /etc/httpd/conf.d/
nano -K zmyapp_info.conf
<VirtualHost *:443>
ServerName info.limour.top
<Location />
AllowOverride AuthConfig
AuthType Basic
AuthName "This page is private"
AuthBasicProvider file
AuthUserFile /media/app/apasswd
require valid-user
</Location>
ProxyPreserveHost On
ProxyRequests Off
ProxyPass / http://127.0.0.1:30101/
ProxyPassReverse / http://127.0.0.1:30101/
</VirtualHost>
systemctl restart httpd



cd /etc/httpd/conf.d/
nano -K zmyapp_frp.conf
<VirtualHost *:443>
ServerName frp.limour.top
ProxyPreserveHost On
ProxyRequests Off
ProxyPass / http://127.0.0.1:11750/
ProxyPassReverse / http://127.0.0.1:11750/
</VirtualHost>
<VirtualHost *:443>
ServerName www.frp.limour.top
ServerAlias *.frp.limour.top
DocumentRoot /var/www/html
SSLEngine on
SSLProtocol +TLSv1.2 +TLSv1.3
SSLProxyProtocol +TLSv1.2 +TLSv1.3
SSLCipherSuite "EECDH+AES128:EECDH+AES256:+SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:RSA+3DES:!DSS"
SSLProxyCipherSuite "EECDH+AES128:EECDH+AES256:+SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:RSA+3DES:!DSS"
SSLCertificateFile /etc/letsencrypt/live/frp.limour.top/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/frp.limour.top/privkey.pem
RewriteEngine On
RewriteCond %{HTTP:Connection} Upgrade [NC]
RewriteCond %{HTTP:Upgrade} websocket [NC]
RewriteRule /(.*) ws://127.0.0.1:21080/$1 [P,L]
ProxyPreserveHost On
ProxyRequests Off
ProxyPass / http://127.0.0.1:21080/
ProxyPassReverse / http://127.0.0.1:21080/
Header always add Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
</VirtualHost>
systemctl restart httpd

cd /etc/httpd/conf.modules.d/
nano -K 03-http2.conf
LoadModule http2_module modules/mod_http2.so
systemctl restart httpd
apachectl -M  grep http2
cd /etc/httpd/conf.d/
nano -K ssl.conf
Protocols h2 h2c http/1.1
systemctl restart httpd

```