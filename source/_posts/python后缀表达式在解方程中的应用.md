---
title: Python后缀表达式在解方程中的应用
tags:
  - Python
  - 后缀表达式
  - 栈
  - 解方程
  - 逆波兰表达式
id: '496'
categories:
  - - Python练习
date: 2020-12-23 16:11:33
---

```
import re
from fractions import Fraction

class Stack(list):
    def isEmpty(self):
        return self == []
    def peek(self):
        if self == []: return None
        else: return self[-1]
    def size(self):
        return len(self)
    push = list.append
    def pop(self):
        if self == []: return None
        else: return super().pop()

class Polynomial(list):
    def __init__(self, value):
        for item in value: self.append(Fraction(item))
    def add(self, value):
        if len(self) < len(value): self += [Fraction(0)]*(len(value)-len(self))
        for i in range(len(value)): self[i] += value[i]
    def sub(self, value):
        if len(self) < len(value): self += [Fraction(0)]*(len(value)-len(self))
        for i in range(len(value)): self[i] -= value[i]
    def mul(self, value):
        tmp = self.copy()
        size = len(self)
        self.clear()
        self += [Fraction(0)]*(size+len(value)-1)
        for i,item in enumerate(value):
            for j in range(size):
                self[i+j] += tmp[j]*item
    def divn(self, n):
        if type(n) is Polynomial:
            for i in range(len(self)): self[i] /= n[0]
        else:
            _n = Fraction(n)
            for i in range(len(self)): self[i] /= _n
    def __str__(self):
        if self == []:
            return '0'
        elif len(self) == 1:
            return str(self[0])
        elif len(self) == 2:
            return f'({self[1]})x + {self[0]}'
        else: pass

def get_Formula(equation):
    return equation.replace(' ','').split('=')

_ep = re.compile(r'([\+\-\*/()][^\+\-\*/()]+)')
_op = {
    '+': lambda x,y:x.add(y),
    '-': lambda x,y:x.sub(y),
    '*': lambda x,y:x.mul(y),
    '/': lambda x,y:x.divn(y)
}
def _middle2behind(Fma, res, s, e):
    sta = Stack()
    _s = s
    while s < e:
        if Fma[s] in '+-':
            if sta.isEmpty():
                sta.push(Fma[s])
                s += 1
            elif sta.peek() in '*/':
                while sta: res.push(sta.pop())
                sta.push(Fma[s])
                s += 1
            elif sta.peek() in '+-':
                res.push(sta.pop())
                sta.push(Fma[s])
                s += 1
        elif Fma[s] in '*/':
            if sta.isEmpty():
                sta.push(Fma[s])
                s += 1
            elif sta.peek() in '+-':
                sta.push(Fma[s])
                s += 1
            elif sta.peek() in '*/':
                res.push(sta.pop())
                sta.push(Fma[s])
                s += 1
        elif Fma[s] == '(':
            s += 1
            d = _middle2behind(Fma, res, s, e)
            s += d
        elif Fma[s] == ')':
            s += 1
            break
        else:
            res.push(Fma[s])
            s += 1
    while sta: res.push(sta.pop())
    return s-_s
def middle2behind(Formula):
    if Formula.startswith('-'): Formula = '0' + Formula
    expr = _ep.findall(Formula.replace('(-','(0-'))
    res = Stack()
    _middle2behind(expr, res, 0, len(expr))
    return res
def str2polynomial(_str):
    if _str.endswith('x'):
        if _str == 'x': return Polynomial((0, 1))
        return Polynomial((0, _str.rstrip('x')))
    else: return Polynomial((_str,))
def not_eval(Formula):
    expr = middle2behind(Formula)
    #print(expr)
    sta = Stack()
    for item in expr:
        if item in _op:
            y = sta.pop()
            x = sta.pop()
            _op[item](x, y)
            sta.push(x)
        else: sta.push(str2polynomial(item))
        #print(sta)
    return sta.pop()

_x = re.compile(r'[a-zA-Z]+')
def solve_eq(equation):
    print(f'equation is \t\t{equation}')
    xname = set(_x.findall(equation))
    if len(xname) != 1: 
        #print(xname)
        print(f'别逗，{equation}是一元一次方程吗?')
        return
    xname = xname.pop()
    Fma = get_Formula(_x.sub('x', equation))
    if len(Fma) != 2:
        print('{equation}不是标准方程！')
        return
    Fma = [not_eval(expr) for expr in Fma]
    print(f'Simplification is \t{Fma[0]} = {Fma[1]}')
    tmp = Fma[0]
    tmp.sub(Fma[1])
    print(f'Transposition is \t{tmp} = 0')
    if len(tmp) < 2 or tmp[1] == 0:
        print(f'{xname} 无解')
    else:
        x = -tmp[0]/tmp[1]
        print(f'{xname} = x = {x}')

```