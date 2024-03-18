---
title: Alist美化：添加评论系统
tags: []
id: '2134'
categories:
  - - uncategorized
date: 2023-02-08 05:08:41
---

搭建完文件分享站后，如果加一个评论系统可以让站点更具互动性，配置好邮件通知也能快速发现失效资源，以及方便老司机们互相交流。

## Demo展示

Demo：[https://od.limour.top/](https://od.limour.top/)

![](https://img.limour.top/archives_2023/2023/02/08/63e32caf028fb.jpg)

## 添加评论系统

先按[Waline](https://waline.js.org/)的官方教程搭建好评论系统，然后在管理面板的元信息的Markdown内容中加入以下内容并应用到子文件夹：

```markdown
#  <center> - 评论 Comments -
<div id="waline"></div>
<script type="module">
    import { init } from 'https://unpkg.com/@waline/client@v2/dist/waline.mjs';
    init({
      el: '#waline',
      serverURL: 'https://comments.limour.top',
    });
</script>
```

## 其他美化

### 设置-全局-自定义头部

```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=String.prototype.replaceAll"></script>
<link rel="stylesheet" href="https://unpkg.com/@waline/client@v2/dist/waline.css" />
<link rel="stylesheet" href="https://npm.elemecdn.com/lxgw-wenkai-webfont@1.1.0/lxgwwenkai-regular.css">
```

### 设置-全局-自定义内容

```html
<div style="text-align:center;background-color: rgba(255,255,255,.3);">
本站当前在线人数 <span style="color: red" id="online_user"></span> 人  
你的在线总时间:  <span style="color: red" id="online_me"></span> 
全站在线总时间:  <span style="color: red" id="online_total"></span>
</div>
<script defer="defer">
(function () {
getOnlineUser()
function getOnlineUser() {
    // 移除之前的 online-counter
    let oldScript = document.getElementById("online-counter");
    if (oldScript) {
        oldScript.remove();
    }
    //create script
    let script = document.createElement("script");
    script.src = "https://time-counter.onmicrosoft.cn/counter.js";
    script.async = true;
    script.id = "online-counter";
    script.setAttribute("interval", 240);
    script.setAttribute("api", "https://time-counter.onmicrosoft.cn/counter");
    script.setAttribute("room", "limourbg");
    document.head.appendChild(script);
}   
})()
</script>
<style type="text/css">
* {font-family: LXGW WenKai;font-weight: bold;}
.notify-render .hope-close-button{display: none;}
.hope-ui-light {background-image: url(https://img.limour.top/archives_2023/2023/02/05/63dece3c51856.webp) !important; background-repeat: no-repeat; background-size: cover; background-attachment: fixed; background-position-x: center;}
.obj-box.hope-stack.hope-c-dhzjXW.hope-c-PJLV.hope-c-PJLV-igScBhH-css {background-color: rgba(255, 255, 255, 0.5) !important;}
.hope-c-PJLV.hope-c-PJLV-ikSuVsl-css {background-color: rgba(255, 255, 255, 0.5) !important;}
.hope-ui-light pre {background-color: rgba(255, 255, 255, 0.1) !important;}
.hope-c-PJLV-ijgzmFG-css {background-color: rgba(255, 255, 255, 0.5) !important;}
</style>
```