---
title: 【从零开始配置VPS】04：WordPress
tags: []
id: '2099'
categories:
  - - uncategorized
date: 2023-01-03 16:00:30
---

## 第一步 拉取镜像

```yml
version: '3.1'
services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8081:80
    environment:
      WORDPRESS_DB_HOST: 172.17.0.1
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./www:/var/www/html
```

*   mkdir -p ~/app/WordPress && cd ~/app/WordPress && nano docker-compose.yml

*   sudo docker-compose up -d

*   反代 8081

## 第二步 创建数据库

*   cd ~/app/WordPress/www

*   wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1.php -O <token>.php

*   访问 limour.top/<token>.php

*   **服务器**：172.17.0.1

*   **用户名**：root

*   **密码**：ROOT\_ACCESS\_PASSWORD

### **第一步，点击“创建新数据库”进行创建数据库**

*   左侧输入框填写数据库名字 wordpress

*   右边的编码选择 utf8mb4\_general\_ci

### **第二步，创建访问该数据库的用户**

*   创建好数据库之后，默认进入该数据库的表和视图，在该界面点击“权限”链接

*   在此界面点击“创建用户”链接

*   权限 勾选 All privileges

*   用户名 wordpress

*   密码 examplepass

## 第三步 WordPress初始化

*   访问 limour.top

*   按提示完成初始化

## 第四步 优化小细节

```php
if(isset($_SERVER['HTTP_X_REAL_IP'])) {
    $list = explode(',',$_SERVER['HTTP_X_REAL_IP']);
    $_SERVER['REMOTE_ADDR'] = $list[0];
    $_SERVER['HTTPS']='on';   
    $_SERVER["SERVER_PORT"] = 443;
    define('WP_HOME', 'https://'.$_SERVER['HTTP_HOST']);
    define('WP_SITEURL', 'https://'.$_SERVER['HTTP_HOST']);
    define('WP_CONTENT_URL', 'https://'.$_SERVER['HTTP_HOST'].'/wp-content');
    define('FORCE_SSL_LOGIN', true);
    define('FORCE_SSL_ADMIN', true);
}
```

*   nano ~/app/WordPress/www/wp-config.php

*   添加以上代码

## 第五步 安装实用插件

*   Easy WP SMTP

*   WP-Optimize

## 第六步 安装Sakurairo主题

*   [下载主题文件](https://gitee.com/mirai-mamori/Sakurairo/repository/archive/main.zip)

*   按[说明](https://iro.tw/)完成Sakurairo安装

### 解除php文件上传的大小限制

*   sudo docker exec -it wordpress-wordpress-1 bash

*   php -i grep php.ini

*   ls /usr/local/etc/php

*   exit

*   cd ~/app/WordPress

*   sudo docker cp wordpress-wordpress-1:/usr/local/etc/php/php.ini-production ./php.ini

*   nano ./php.ini

*   upload\_max\_filesize = 20M

*   post\_max\_size = 20M

*   sudo docker cp ./php.ini wordpress-wordpress-1:/usr/local/etc/php/php.ini

*   sudo docker-compose restart