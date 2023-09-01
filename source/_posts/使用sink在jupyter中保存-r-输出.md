---
title: 使用sink在Jupyter中保存 R 输出
tags: []
id: '1298'
categories:
  - - 有趣技能
date: 2021-12-11 00:32:59
---

## 背景

Jupyter的ir本身使用来sink来获得输出，因此用sink捕获的代码必须与sink在一行

## 定义不会分次执行的sink

```r
f_sink <- function(code, log_path='out.log'){
    msgcon <- file(log_path, open = "a")
    sink(msgcon, append=TRUE)
    sink(msgcon, append=TRUE, type="message")
    print(paste(Sys.time(),'sink start'))

    code()
    
    print(paste(Sys.time(),'sink end'))
    sink() 
    sink(type="message")
    close(msgcon)
}
```

## 通过定义函数来绕过ir的分行执行

```r
f_code_1 <- function(){
    print('hello world1')
    
    x <<- rnorm(100,0,1) 
    print(mean(x))
    
    print('hello world2')
}

f_sink(f_code_1)
print(x)
```