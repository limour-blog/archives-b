---
title: ggplot配色修改方法
tags: []
id: '1964'
categories:
  - - uncategorized
date: 2022-08-10 23:34:08
---

获得一个好看的配色可以通过一些专业配色的网站，比如[chinavid](https://www.chinavid.com/webcolor.html)，或者从ggsci包里挑选。颜色选好了，如果是ggsci包的话好办，直接scale\_color\_xxx或scale\_fill\_xxx就行，但是自己选的配色要怎么调呢？

*   对于二色梯度，可以使用scale\_colour\_gradient和scale\_fill\_gradient
*   对于三色梯度，可以使用scale\_colour\_gradient2和scale\_fill\_gradient2
*   对于N色梯度，可以使用scale\_colour\_gradientn和scale\_fill\_gradientn
*   对于离散的映射，可以使用scale\_color\_manual和scale\_fill\_manual
*   使用RColorBrewer调色板，可以用scale\_color\_brewer和scale\_fill\_brewer
*   使用灰度图，则用scale\_color\_grey和scale\_fill\_grey
*   使用调色板做梯度，有scale\_color\_distiller和scale\_fill\_distiller