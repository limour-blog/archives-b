---
title: NPM面板：反代grpc连接
tags: []
id: '1884'
categories:
  - - uncategorized
date: 2022-07-11 05:33:30
---

*   在~/ngpm/data/nginx/proxy\_host下找到对应的conf文件
*   添加以下代码
*   后续使用NPM后台面板上修改原来的配置需要重复以上步骤

```Nginx
location /serviceName {
  if ($content_type !~ "application/grpc") {
    return 404;
  }
  client_max_body_size 0;
  client_body_timeout 1071906480m;
  grpc_read_timeout 1071906480m;
  grpc_pass grpc://docker0:port;
}
```