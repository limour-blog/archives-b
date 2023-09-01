---
title: Itertools之groupby的使用
tags:
  - groupby
  - Itertools
  - Python
id: '119'
categories:
  - - Python练习
date: 2020-05-06 16:36:31
---

```
'''
随机生成一批学生的学号（6位整数）和成绩[0,100]，完成：
按成绩升序排列；
统计各区间[0,60),[60,70),[70,80),[80,90),[90,100]的人数；
列出各区间的名单（按学号降序）。


参考结果如下页：

名单和成绩已生成完毕。

按成绩排序：
79017128
34497263
38093966
89848868
60695168
60591873
43285580
29561281
27561393
459370100

各个区间的人数：
0-60:1人
60-70:4人
70-80:1人
80-90:2人
90-100:2人

各个区间的名单：
0-60:790171
60-70:898488 606951 380939 344972
70-80:605918
80-90:432855 295612
90-100:459370 275613
'''

from random import randint
from  itertools import groupby

students = {randint(100000,999999):randint(0,100) for i in range(randint(10,20))}
#生成[10,20]个学生及其成绩,字典可以防止sId重复,因此学生数可能少于10
print('名单和成绩已生成完毕。')

print('\n按成绩排序：')
sl = sorted(students.items(),key = lambda x:x[1])#按成绩排序(sorted list)
print(*[f'{sId}{scores}' for sId,scores in sl], sep='\n')

gs = list(range(60, 100, 10))#分组标准 (grouping standards)
dgs = [f'{start}-{end}' for start,end in zip([0]+gs, gs+[100])]
#生成分组标准的描述 (description of grouping standards ) = ['0-60', '60-70', '70-80', '80-90', '90-100']

g = [(dgs[i],list(v)) for i,v in groupby(sl, lambda x: sum(x[1] >= st for st in gs))]
assert sum(len(sg) for d,sg in g) == len(students)#调试用

print('\n各个区间的人数：')
print(*[f'{d}:{len(sg)}人' for d,sg in g], sep='\n')

```