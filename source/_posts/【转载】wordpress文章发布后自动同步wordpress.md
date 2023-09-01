---
title: 【转载】WordPress文章发布后自动同步wordpress
tags: []
id: '1874'
categories:
  - - 转载
date: 2022-07-09 23:11:42
---

## 来源

[https://blog.csdn.net/u012241616/article/details/110563637](https://blog.csdn.net/u012241616/article/details/110563637)

思路是：在另一个 WordPress 站点创建一个API，文章发布时用cURL模拟 POST 请求 API 利用wp\_insert\_post()函数来创建文章。支持同步文章标题、内容、类型。

## 同步站点的设置

在同步博客站点的根目录创建一个文件，命名为GNRq3qU3Fjs4qA3m.php，代码如下：

```php
<?php  
//文章接收  
define('WP_USE_THEMES', false);  
require_once("wp-load.php");  
$key='6K3DntjLzGNzqsh4'; 
if($_POST['key']==$key){  
    $info = array(  
    'post_title' => $_POST['title'],  
    'post_content' => $_POST['content'],  
    'post_status' => 'publish',  
    'post_author' => 1, //发布文章的作者ID，1 为管理员  
    'post_date' => $_POST['date'],  
    'post_type' => $_POST['type']  
    );  
    wp_insert_post( $info );  
}
wp_mail('limour@aliyun.com', "post sync", json_encode($_POST, JSON_UNESCAPED_UNICODE));
echo 'hello world';
```

## 源站点的设置

复制下面的代码，用 Notepad ++等工具另存为 php 文件，打包成 zip 文档，在 wordpress 插件安装后台上传，安装并启用。

```php
<?php
/*
Plugin Name: 文章推送
Plugin URI: https://blog.csdn.net/u012241616/article/details/110563637
Description: 自用的文章同步插件
Version: 1.0
Author: 璃墨Limour
Author URI: https://blog.j11.fun/
License: GPL2
*/
function fanly_sync_post($post_ID) {  
    $key='6K3DntjLzGNzqsh4'; //输入你设置的密钥  
    $url='https://wp.j11.fun/GNRq3qU3Fjs4qA3m.php';//API地址,就是接受数据的那个站点
    $post_info = get_post($post_ID);  
    if ( $post_info->post_status == 'publish' && $_POST['original_post_status'] != 'publish' ) {  
        $title=$post_info->post_title;  
        //wp_mail('limour@aliyun.com', "fanly_sync_post", '同步标题'.json_encode($post_info, JSON_UNESCAPED_UNICODE));
        $content=$post_info->post_content;
        $date=$post_info->post_modified_gmt;  
        $type=$post_info->post_type;
        $data=array ("key"=>$key,"title"=>$title,"content"=>$content,"date"=>$date,"type"=>$type);
        $ch = curl_init (); //cURL模拟POST  
        curl_setopt ( $ch, CURLOPT_RETURNTRANSFER, TRUE );  
        curl_setopt ( $ch, CURLOPT_POST, TRUE );  
        curl_setopt ( $ch, CURLOPT_POSTFIELDS, $data );  
        curl_setopt ( $ch, CURLOPT_URL, $url );  
        curl_setopt ( $ch, CURLOPT_SSL_VERIFYPEER, TRUE);  
        $ret = curl_exec ( $ch );  
        curl_close ( $ch );  
        return $ret;  
   }
}
add_action('publish_post', 'fanly_sync_post'); //钩子，在文章发布时执行  
?>
```

## 最终效果

[https://wp.j11.fun/](https://wp.j11.fun/)