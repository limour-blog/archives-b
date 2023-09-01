---
title: 让你反代的openai的API支持key轮询
tags: []
id: '2179'
categories:
  - - uncategorized
date: 2023-03-26 03:55:21
---

[利用Golang反代某AI的API](https://occdn.limour.top/2607.html)中的反代只能用一个key，而前端项目那么多，总是改前端的代码让其支持轮询太麻烦了，干脆改造反代的代码，将轮询写到反代的环节中。

```golang
package main

import (
"log"
"net/http"
"net/http/httputil"
"net/url"
"strings"
"sync"
)

type RProxy struct {
remote *url.URL
}

var (
    mu    sync.Mutex
    count int
)

func get1Key(key string) string {
    mu.Lock()
    defer mu.Unlock()

    arr := strings.Split(key, "")
    randomIndex := count % len(arr)
    count++
    if count > 999999 {
        count = 0
    }
    randomSubstr := arr[randomIndex]
    return randomSubstr
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
keys := strings.Split(request.Header.Get("Authorization"), " ")
if len(keys) == 2 {
request.Header.Set("Authorization", "Bearer " + get1Key(keys[1]))
}
log.Println("request.Header.Authorization：", request.Header.Get("Authorization"))
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

*   CC=musl-gcc /home/jovyan/go/bin/go1.20.1 build -tags musl -o openai -trimpath -ldflags '-linkmode "external" -extldflags "-static" -s -w -buildid=' ./openai.go
*   docker build -t limour/openai .
*   sudo docker-compose up -d --remove-orphans
*   sudo docker image prune
*   docker-compose logs tail