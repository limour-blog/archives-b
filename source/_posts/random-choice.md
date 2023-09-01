---
title: random.choice
tags:
  - Python
  - random.choice
id: '129'
categories:
  - - Python练习
date: 2020-05-30 15:30:50
---

```
from decimal import Decimal
import random, re

rd = re.compile(r'^-?\d*(?:\.\d*)?\d$')
Norm = Decimal('0.01')

def randint(a=0, b=10):
    return str(random.randint(a,b))

def _add():
    a, b = randint(), randint()
    return f'{a} + {b} =', Decimal(a) + Decimal(b)

def _sub():
    a, b = randint(), randint()
    return f'{a} - {b} =', Decimal(a) - Decimal(b)

def _mul():
    a, b = randint(), randint()
    return f'{a} * {b} =', Decimal(a) * Decimal(b)

def _dev():
    a, b = randint(), randint(1)
    return f'{a} / {b} =', (Decimal(a) / Decimal(b)).quantize(Norm, rounding = 'ROUND_HALF_UP')

opset = (_add, _sub, _mul, _dev)
def _op():
    return random.choice(opset)()

total = correct = 0
al = []
ct = ''
while True:
    if ct != 'y':
        expression, answer = _op()
    else:
        ct = ''
    ua = input(f'第{total+1}题:  {expression} ').replace(' ','')
    if rd.match(ua):
        total += 1
        if Decimal(ua).quantize(Norm, rounding = 'ROUND_HALF_UP') == answer:
            correct += 1
            al.append(f'第{total}题: {expression} {ua} 正确！')
            #print("此题回答正确！" )
        else:
            al.append(f'第{total}题: {expression} {ua} 错误！标准答案是 {answer}')
            #print(f"此题回答错误！标准答案是 {answer}" )
    else:
        while True:
            ct = input('输入错误，是否继续(y/n)').strip().lower()
            if ct in 'yn':
                break
        if ct == 'n':
            break

```