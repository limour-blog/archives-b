---
title: 深入认识eval
tags:
  - eval
  - JSONDecodeError
  - Python
id: '151'
categories:
  - - Python练习
date: 2020-06-19 00:06:57
---

最近在处理服务器返回的json字串时出现了报错:  
`JSONDecodeError: Expecting property name enclosed in double quotes`  
原因是这个json字串是非标准的,里面的属性名称不带双引号,导致python无法识别  
经过一番搜索找到这样一个函数可以处理这个问题:

```Python
def jsonfy(s:str)->object:
#此函数将不带双引号的json的key标准化
obj = eval(s, type('js', (dict,), dict(getitem=lambda s, n: n))())
return obj
```

```
看上去一脸懵逼,其实原理很好懂,主要是要知道eval函数的定义中,第二个可选参数是eval运行环境的全局变量:
eval(source, globals=None, locals=None, /)
    Evaluate the given source in the context of globals and locals.

    The source may be a string representing a Python expression
    or a code object as returned by compile().
    The globals must be a dictionary and locals can be any mapping,
    defaulting to the current globals and locals.
    If only globals is given, locals defaults to it.
直接eval会报NameError,变量未定义,那么我们只要让所有的变量都定义为它的变量名字串就好了
所以用type写一个dict的子类,修改__getitem__,将所有键的值定义为键的字串,然后用这个替换eval的命名空间dict就可以了
是不是很妙呢?感觉各种json库都可以不使用了哈哈哈
```