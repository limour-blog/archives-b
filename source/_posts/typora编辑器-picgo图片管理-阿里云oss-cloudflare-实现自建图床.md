---
title: Typora编辑器 + PicGo图片管理 + 阿里云OSS +  cloudflare 实现自建图床
tags: []
id: '1658'
categories:
  - - 建站探索
date: 2022-02-17 00:37:18
---

## 第一步 Typora中启用PicGo.app

*   文件-偏好设置-图像-上传服务
*   下载安装PicGo.app并启用

## 第二步 新建OSS Bucket，并配置专用管理子账户

*   https://oss.console.aliyun.com/overview
*   创建和阿里云服务器同区域的Bucket
*   权限管理-**防盗链** 设置好Referer；**注意：必须带上https://，如https://\*.limour.top**
*   权限管理-**访问控制RAM** 设置好子账号，访问方式为 "Open API 调用访问"
*   RAM 访问控制-身份管理-用户-添加权限，给予 "AliyunOSSFullAccess" 权限

## 第三步 cloudflare开启CDN

*   新建A记录 指向阿里云服务器的IP
*   确保开启小云朵

## 第四步 使用阿里云服务器反代OSS资源，避免外网流量

*   反代域名选用"ECS 的经典网络访问"或"ECS 的 VPC 网络访问"的"Bucket 域名"
*   以下是apache反代的示例

```apache
<VirtualHost *:443>
ServerName img-cdn.limour.top
ProxyPreserveHost Off
ProxyRequests Off
ProxyPass / http://limour-img.oss-cn-shanghai-internal.aliyuncs.com/
ProxyPassReverse / http://limour-img.oss-cn-shanghai-internal.aliyuncs.com/
</VirtualHost>
```

## 第五步 正常配置PicGo

*   设置自定义域名填cloudflare代理的域名 https://img-cdn.limour.top
*   Typora中验证图片上传选项

## 第六步 开放指定ip的匿名读取权限

*   Bucket-权限管理-**Bucket 授权策略** 指定ip为自己的阿里云服务器
*   **注意：该ip须为内网ip，不能是公网ip，一般为 172 开头**

![授权页面](https://img-cdn.limour.top/20220217004941.png)