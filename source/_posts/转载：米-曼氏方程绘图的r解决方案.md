---
title: 转载：米-曼氏方程绘图的R解决方案
tags: []
id: '1197'
categories:
  - - R绘图
date: 2021-11-01 04:02:59
---

> [米-曼氏方程绘图的R解决方案](https://mp.weixin.qq.com/s/9_u21w1DN_FayhVFfvuznQ) @**桢和他的朋友们**
> 
> 1 #换一种数据组合方式，增加分组列，便于用colour或shape表示不同组  
> 2 biochem\_1 = data.frame(  
> 3  concen = c(1/concen,1/concen),  
> 4  velocity = c(1/velocity,1/velocity\_inhibit),  
> 5  gruops = c(rep("control",8),rep("inhibited",8))  
> 6 )#已经取好倒数了  
> 7  
> 8 #画的时候美化一下，注意pgeom\_point直接使用ggplot中的mapping即可。  
> 9 ggplot(data = biochem\_1,aes(x=concen,y=velocity,colour = gruops))+  
> 10  geom\_point(size=3,alpha=0.8)+  
> 11  geom\_abline(intercept = fitted\_model$coefficients\[\[1\]\],  
> 12              slope = fitted\_model$coefficients\[\[2\]\],  
> 13              size=1,color="lightgray")+  
> 14  geom\_abline(intercept = fitted\_model\_inhibit$coefficients\[\[1\]\],  
> 15              slope = fitted\_model\_inhibit$coefficients\[\[2\]\],  
> 16              size=1,color="lightgray")+  
> 17  scale\_x\_continuous(limits = c(-0.4,1.6))+  
> 18  scale\_y\_continuous(limits = c(-0.005,0.11))+  
> 19  labs (title="酶促反应动力学实验",x="1/\[S\]",y="1/v")+  
> 20  theme\_light()+  
> 21  theme(  
> 22    plot.title = element\_text(hjust = 0.5),  
> 23    legend.background = element\_roundrect(fill=NA,r=0.15,color = "lightgray")  
> 24  ) +  
> 25  geom\_hline(aes(yintercept=0))+  
> 26  geom\_vline(aes(xintercept=0))+  
> 27  scale\_colour\_discrete(name  ="组别",  
> 28                        breaks=c("control", "inhibited"),  
> 29                        labels=c("对照组", "抑制剂组"))

## 第一步 导入数据

```r
data = read.table(header = T, row.names = 1,
text = '
序号 浓度倒数 无抑制剂 有抑制剂
11.60000 0.05607 0.12833 
21.40000 0.07911 0.08462 
31.20000 0.05848 0.06696 
41.00000 0.04894 0.06979 
50.80000 0.03080 0.06095 
60.60000 0.02658 0.05168 
70.40000 0.02104 0.03479 
80.20000 0.02183 0.02552 
')
```

## 第二步 拟合直线

```r
`无抑制剂` = lm(`无抑制剂`~`浓度倒数`,data = data[-c(2,3,8),])
`有抑制剂` = lm(`有抑制剂`~`浓度倒数`,data = data[-3,])
s3 <- summary(`无抑制剂`)
s4 <- summary(`有抑制剂`)
```

## 第三步 绘图

```r
f_gp_xyn <- function(df, xix=1){
    if(is.character(xix)){
        xix = which(colnames(df) == xix)
    }
    lc_x = rep(df[,xix], ncol(df) - 1)
    lc_y = NULL
    lc_g = NULL
    lc_n = nrow(df)
    for (lc_c in colnames(df)[-xix]){
        lc_y = c(lc_y, df[, lc_c])
        lc_g = c(lc_g, rep(lc_c, lc_n))
    }
    data.frame(x = lc_x, y = lc_y, g = as.factor(lc_g))
}
```

*   简易版，不支持移除无效点

```r
options(repr.plot.width=12, repr.plot.height=12)
options(ggrepel.max.overlaps = Inf)
require(ggplot2)

ggplot(data = f_gp_xyn(data),aes(x=x,y=y,colour = g, shape = g))+
geom_point(size=3,alpha=0.8) + 
stat_smooth(method = 'lm', se = FALSE)
```

[![](https://img-cdn.limour.top/blog_wp/2021/11/下载-1.png)](https://img-cdn.limour.top/blog_wp/2021/11/下载-1.png)

*   正式版

```r
options(repr.plot.width=12, repr.plot.height=12)

ggplot(data = f_gp_xyn(data),aes(x=x,y=y,colour = g, shape = g))+
geom_point(size=3,alpha=0.8) + 
geom_abline(intercept = s3$coefficients[[1]], slope = s3$coefficients[[2]], linetype="dashed")+ 
geom_abline(intercept = s4$coefficients[[1]], slope = s4$coefficients[[2]], linetype="dashed")+
scale_x_continuous(limits = c(-0.4,1.6), breaks=seq(-0.4, 1.6, 0.1))+
scale_y_continuous(limits = c(0,0.14), breaks=seq(0, 0.14, 0.005))+
labs (title="Lineweaver-Burk equation",x="1/[S]",y="1/v")+
theme_light() +
theme_bw(base_size=18) +
theme(plot.title = element_text(hjust = 0.5),
    axis.text.y = element_text(size = 12),
    axis.text.x = element_text(size = 12, angle=90))+
geom_hline(aes(yintercept=0))+geom_vline(aes(xintercept=0))+
scale_shape_discrete(name="group",
    labels=c("control", "inhibited"))+
scale_color_discrete(name="group",
    labels=c("control", "inhibited"))

ggsave("example2.pdf")
```

[![](https://img-cdn.limour.top/blog_wp/2021/11/5.png)](https://img-cdn.limour.top/blog_wp/2021/11/5.png)