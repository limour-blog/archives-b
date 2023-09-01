---
title: Maxima辅助检查高数作业
tags:
  - Maxima
  - 高数
id: '126'
categories:
  - - 软件分享
date: 2020-05-16 15:03:41
---

```
a:0;b:1;p:1;q:1;
v1(x):=a1*x+a2;
v2(x):=b1*x+b2;
y(x) := v1(x)*e^(a*x)*sin(b*x)+v2(x)*e^(a*x)*cos(b*x);
t(x):=diff(y(x),x,2)+p*diff(y(x),x,1)+q*y(x);
u(x):=subst(1,log(e),ratsimp(t(x)/e^(a*x)));
u(x);
linsolve([a1=0,b1+a2+2*a1=0,-b1=1,-b2-2*b1+a1=0],[a1,b1,a2,b2]);
```