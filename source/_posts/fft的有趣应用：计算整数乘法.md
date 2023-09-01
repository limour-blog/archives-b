---
title: FFT的有趣应用：计算整数乘法
tags: []
id: '2088'
categories:
  - - uncategorized
date: 2022-12-23 09:24:44
---

今天刷知乎，看到了一个有趣的回答，来验证一下

## 回答摘录

[怎么判断一个孩子有没有数学天赋？](https://www.zhihu.com/question/543229591/answer/2760302705)

[间宫羽咲sama](https://www.zhihu.com/people/jian-gong-yu-xiao-sama)的回答以及一些有趣的评论

抖个机灵

考他34×77=?

如果他不说解是什么，只告诉你解一定是一个有界的常数，那么他适合搞纯数学。

如果他偷偷拿出计算器算出结果，然后不给过程只说结果是2618，那么他适合搞工科。

如果他说对{4,3,0,0}和{7,7,0,0}做FFT得到{7,4-3i,1,4+3i}和{14,7-7i,0,7+7i}，相乘后逆FFT得到{28,49,21,0}，加权相加得到2618，并给你证明了用FFT算法计算大数乘法复杂度为O(n logn loglogn)优于平方复杂度的竖式乘法，那么恭喜你，你的孩子起码有八核。

评论1：34x77=2618 如果结果是1010 0011 1010，那么恭喜你，你家的孩子一定是x16位以上计算机系统

评论2：等于8234，我确定数量级是对的。恭喜你的孩子可以当天体物理学家。

评论3：如果回答你2000，恭喜你这是个实验物理奇才

评论4：有人……啊，问了我这么一个问题，嗯，问我……34\*77是多少啊，我说，你们都知道是多少，啊，但是，为什么我要说这个（喝水，呸），我们说，工作，不是简单的加法，而是什么？啊？我问问你们，是什么？对，乘法！如果说，一位同志奉献了34，啊，那么，那么，注意啊，我们77位同志，该是多少啊？这是非~常大的能量，这个就是乘法精神！我们……不好意思我接个电话

评论5：不知道34×77结果是多少，但是发现在（34-δφ）（77+77/34δφ）对称性下结果不变，那就适合去当理论物理学家

评论6：如果他说2618小于2000，可以当个经济学家

评论7：如果他会从这条算式中萌生找出任意一个数的所有4n+1型素因子的想法的话，那他可能适合搞数论

评论8：闭上眼大概七八秒之后说2618，适合考试

评论9：如果说你家孩子发现了34和77之间的这种运算能满足交换律结合律，在R上封闭，甚至不仅仅34和77有这结论还能推广到任意其他的两个R上的元素上，还发现R上这种运算存在单位元，甚至这俩任意元素在另一种被记为“+”的算子下也满足交换律结合律居然有单位元有零元有逆元，甚至能证明“+”和你的“\*”居然满足分配率，那恭喜你家的孩子居然独立证明了R是一个线性空间，日后一定是个代数天才

评论10：孩子找了几个老师家长同学都问了一下，然后算了个均值，那么她适合搞统计

评论11：如果是他用77÷3，算出约等于26，再添2个0，说明他适合当公务员

## 配置环境

*   conda create -n scipy -c conda-forge scipy -y
*   conda activate scipy
*   conda install -c conda-forge ipykernel -y
*   python -m ipykernel install --user --name scipy

## 回答验证

```python
import numpy as np
from scipy.fftpack import fft, ifft
def nextPower2(L):
    return np.power(2,np.ceil(np.log2(L))).astype(int)
def int2Array(n, size):
    res = np.zeros(nextPower2(size), dtype = np.int8)
    i = 0
    while n > 0:
        n, res[i] = divmod(n, 10)
        i += 1
    return res
def array2Int(arr):
    return np.dot(np.around(arr,0),10**np.arange(len(arr)))
```

```python
a = fft(int2Array(34, 4))
b = fft(int2Array(77, 4))
c = a * b
d = ifft(c)
e = array2Int(d)
a,b,c,d,e
# (array([7.-0.j, 4.-3.j, 1.-0.j, 4.+3.j]),
#  array([14.-0.j,  7.-7.j,  0.-0.j,  7.+7.j]),
#  array([98. -0.j,  7.-49.j,  0. -0.j,  7.+49.j]),
#  array([28.+0.j, 49.+0.j, 21.-0.j,  0.+0.j]),
#  (2618+0j))
 
array2Int(ifft(fft(int2Array(457, 6))*fft(int2Array(756, 6))))
# (345492+0j)
```