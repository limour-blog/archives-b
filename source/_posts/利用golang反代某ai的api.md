---
title: 利用Golang反代某AI的API
tags: []
id: '2155'
categories:
  - - uncategorized
date: 2023-03-03 14:24:04
---

某AI的API在部分地区不太稳定，这里可以在稳定的地区进行反代。

## 反代用的Golang源码

```golang
package main

import (
"log"
"net/http"
"net/http/httputil"
"net/url"
"strings"
)

type RProxy struct {
remote *url.URL
}

func GoReverseProxy(this *RProxy) *httputil.ReverseProxy {
remote := this.remote

proxy := httputil.NewSingleHostReverseProxy(remote)

proxy.Director = func(request *http.Request) {
targetQuery := remote.RawQuery
request.URL.Scheme = remote.Scheme
request.URL.Host = remote.Host
request.Host = remote.Host // todo 这个是关键
request.URL.Path, request.URL.RawPath = joinURLPath(remote, request.URL)

if targetQuery == ""  request.URL.RawQuery == "" {
request.URL.RawQuery = targetQuery + request.URL.RawQuery
} else {
request.URL.RawQuery = targetQuery + "&" + request.URL.RawQuery
}
if _, ok := request.Header["User-Agent"]; !ok {
// explicitly disable User-Agent so it's not set to default value
request.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36")
}
log.Println("request.URL.Path：", request.URL.Path, "request.URL.RawQuery：", request.URL.RawQuery)
}

// 修改响应头
proxy.ModifyResponse = func(response *http.Response) error {
response.Header.Add("Access-Control-Allow-Origin", "*")
response.Header.Add("Reverse-Proxy-Server-PowerBy", "(Hzz)https://hzz.cool")
return nil
}

return proxy
}

// go sdk 源码
func joinURLPath(a, b *url.URL) (path, rawpath string) {
if a.RawPath == "" && b.RawPath == "" {
return singleJoiningSlash(a.Path, b.Path), ""
}
// Same as singleJoiningSlash, but uses EscapedPath to determine
// whether a slash should be added
apath := a.EscapedPath()
bpath := b.EscapedPath()

aslash := strings.HasSuffix(apath, "/")
bslash := strings.HasPrefix(bpath, "/")

switch {
case aslash && bslash:
return a.Path + b.Path[1:], apath + bpath[1:]
case !aslash && !bslash:
return a.Path + "/" + b.Path, apath + "/" + bpath
}
return a.Path + b.Path, apath + bpath
}

// go sdk 源码
func singleJoiningSlash(a, b string) string {
aslash := strings.HasSuffix(a, "/")
bslash := strings.HasPrefix(b, "/")
switch {
case aslash && bslash:
return a + b[1:]
case !aslash && !bslash:
return a + "/" + b
}
return a + b
}

func main() {
port := "1874"
reverseUrl := "https://api.openai.com"

remote, err := url.Parse(reverseUrl)
if err != nil {
panic(err)
}

proxy := GoReverseProxy(&RProxy{
remote: remote,
})

log.Println("当前代理地址： " + reverseUrl + " 本地监听： http://127.0.0.1:" + port)

serveErr := http.ListenAndServe(":"+port, proxy)

if serveErr != nil {
panic(serveErr)
}
}
```

## 编译Golang源码

*   [尝试用Golang静态编译可执行文件](https://occdn.limour.top/2599.html)
*   CC=musl-gcc /home/jovyan/go/bin/go1.20.1 build -tags musl -o openai -trimpath -ldflags '-linkmode "external" -extldflags "-static" -s -w -buildid=' ./openai.go

## 编写Dockerfile

```Dockerfile
# set alpine as the base image of the Dockerfile
FROM alpine:latest
 
# Copy over the torrc created above and set the owner to `tor`
COPY openai /bin/openai
 
# Set `tor` as the entrypoint for the image
ENTRYPOINT ["/bin/openai"]
```

*   mkdir -p ~/app/openai && cd ~/app/openai && nano Dockerfile && nano docker-compose.yml
*   上传编译好的openai可执行文件
*   chmod +x openai
*   docker build -t limour/openai .

## 部署Docker并反代

```yml
version: '3.3'
services:
    openai:
        container_name: openai
        restart: always
        image: limour/openai
    
networks:
  default:
    external: true
    name: ngpm
```

*   [Nginx Proxy Manager进阶：使用Docker network](https://occdn.limour.top/2523.html)
*   sudo docker-compose up -d && sudo docker-compose logs
*   然后在Nginx Proxy Manager中反代openai:1874