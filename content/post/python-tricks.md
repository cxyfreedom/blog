---
title: Python 中的一些技巧
date: 2019-01-10T18:08:40+08:00
description: ""
tags: ["Python", "Tricks"]
categories: ["Python"]
images:
---
本文为阅读 《Python Tricks A Buffet of Awesome Python Features》书籍的笔记，仅供自己参考。由于大部分内容熟知，所以并未记录。

<!--more-->

## 1. 使用断言

示例：
```python
def apply_discount(product, discount):
   price = int(product['price'] * (1 - discount))
   assert 0 <= price <= product['price']
   return price


shoes = {'name': 'Fancy shoes', 'price': 14900}
apply_discount(shoes, 0.25)  # correct 
apply_discount(shoes, 2.0)   # incorrect
```

断言是 debug 的好方法，但不是处理运行时发生错误的方式。当 `__debug__` 变量为 `True` 时，assert 语句才会被执行。

assert 语法：
```python
assert expression1 [, expression2]
```

注意事项：

* 不要使用断言进行数据验证
* 不要让断言永远不失败

>断言和异常的区别：检查先验条件使用断言，检查后验条件使用异常

## 2. Context Manager

两种方式实现：`class-based` 和 `generator-based`。

示例1：
```python
class ManagedFile:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()

with ManagedFile('hello.txt') as f:
    f.write('hello, world!')
```

示例2:
```python
from contextlib import contextmanager

@contextmanager
def managed_file(name):
    try:
        f = open(name, 'w')
        yield f
    finally:
        f.close()

with managed_file('hello.txt') as f:
    f.write('hello, world!')
```

## 3. `__repr__`

Python3.X 写法：
```python
class Car:
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __repr__(self):
        return (f'{self.__class__.__name__}('
                f'{self.color!r}, {self.mileage!r})')

    def __str__(self):
        return f'a {self.color} car'
```

Python2.X 写法：
```python
class Car(object):
    def __init__(self, color, mileage):
        self.color = color
        self.mileage = mileage

    def __repr__(self):
        return '{}({!r}, {!r})'.format(self.__class__.__name__, self.color,
                                       self.mileage)

    def __unicode__(self):
        return u'a {self.color} car'.format(self=self)

    def __str__(self):
        return unicode(self).encode('utf-8')
```

* `__str__` 的结果应该是有可读性的；`__repr__` 的结果应该是明确的。
* 在 Class 中总是应该实现 `__repr__`；`__str__` 默认会调用 `__repr__` 的实现。
* 在 Python2 中用 `__unicode__` 代替 `__str__`。

## 4. Namedtuple

示例：
```python
import json
from collections import namedtuple

Car = namedtuple('Car', 'color, mileage')
my_car = Car('read', 3812.4)
print(my_car)

class MyCarWithMethods(Car):
    def hexcolor(self):
        return '#ff0000' if self.color == 'red' else '#000000'

c = MyCarWithMethods('red', 1234)
print(c.hexcolor())

ElectricCar = namedtuple('ElectricCar', Car._fields + ('charge', ))
electric_car = ElectricCar('red', 1234, 45.0)
print(electric_car)

print(my_car._asdict())
print(json.dumps(my_car._asdict()))

my_car._replace(color='blue')
print(my_car)

new_car = Car._make(['red', 999])
print(new_car)
```

## 5. 字典

★ ChainMap

可以将多个不同的字典放在一起进行搜索。插入、更新、删除只会影响第一个添加进 chain 的字典。

```python
dict1 = {'one': 1, 'two': 2}
dict2 = {'three': 3, 'four': 4}
chain = ChainMap(dict1, dict2)
print(chain)
```

★ MappingProxyType

创建一个只读的字典

```python
from types import MappingProxyType

writable = {'one': 1, 'two': 2}
read_only = MappingProxyType(writable)

print(read_only)
# read_only['one'] = 3  # TypeError
writable['one'] = 4
print(read_only)  #  {'one': 4, 'two': 2}
```

★ fromkeys

```python
dict.fromkeys(['name', 'age'], '(unknown)')
```

## 6. 字符串

★ 模板字符串

```python
from string import Template

tmpl = Template("Hello, $who! $what enough for ya?")
res = tmpl.substitute(who="Mars", what="Dusty")
```

★ 宽度、精度和千位分隔符

```python
# 宽度
from math import pi
>>> "{num:10}".format(num=3)
'         3'
>>> "{name:10}".format(name="freedom")
'freedom   '
# 精度
>>> "Pi day is {pi:.2f}".format(pi=pi)
'Pi day is 3.14'
# 千位分隔符
>>> "One googol is {:,}".format(10**10)
'One googol is 10,000,000,000'
# 用 0 填充
>>> "{:010.2f}".format(pi)
'0000003.14'
# 对齐，左对齐、右对齐和居中，可分别使用<、>和^
>>> print('{0:<10.2f}\n{0:^10.2f}\n{0:>10.2f}'.format(pi))
3.14      
   3.14   
      3.14
# 填充字符
>>> "{:$^15}".format(" WIN BIG ")
'$$$ WIN BIG $$$' 
>>> print('{0:10.2f}\n{1:10.2f}'.format(pi, -pi)) 
      3.14
     -3.14
>>> print('{0:10.2f}\n{1:=10.2f}'.format(pi, -pi)) 
      3.14
-     3.14
# 加上正负号
>>> print('{0:+.2}\n{1:+.2}'.format(pi, -pi))
+3.1
-3.1
# 特殊转换
>>> "{:#b}".format(42)
'0b101010'
```

## 7. 类

★ 获取一个类的基类

```python
class Test:
    pass

print(Test.__bases__)  # <class 'object'>
```

一些相关函数

```python
callable(object)  # 判断对象是否可以调用
getattr(object, name[, default])  # 获取属性的值，还可以提供默认值
hasattr(object, name)  # 获取对象是否有指定的属性
setattr(object, name, value)  # 将对象的指定属性设置为指定的值
```

## 8. 魔法方法

* `__len__`: 这个方法返回集合包含的项数。
* `__getitem__(self, key)`: 这个方法返回与指定键相关联的值。
* `__setitem__(self, key, value)`: 这个方法设置与键相关联的值。
* `__delitem__(self, key)`: 删除与 key 相关联的值。
* `__getattribute__(self, name)`: 在属性被访问时自动调用。
* `__getattr__(self, name)`: 在属性被访问而对象没有这样的属性时自动调用。
* `__setattr__(self, name, value)`: 试图给属性赋值时自动调用。
* `__delattr__(self, name)`: 试图删除属性时自动调用。

实现了 `__iter__` 方法的对象是可迭代的，实现了方法 `__next__` 的对象是迭代器。

## 9. fileinput

迭代一系列文本文件中的所有行。

比如如下代码用于给脚本添加行号。
```python
import fileinput

for line in fileinput.input(inplace=False):
    line = line.rstrip()
    num = fileinput.lineno()
    print('{:<50} # {:2d}'.format(line, num))
```

## 10. heapq

相关函数

* `heappush(heap, x)`: 将 x 压入堆中
* `heappop(heap)`: 从堆中弹出最小的元素
* `heapify(heap)`: 让列表具有堆特征
* `heapreplace(heap, x)`: 弹出最小的元素，并将 x 压入堆中
* `nlargest(n, iter)`: 返回 iter 中 n 个最大的元素
* `nsmallest(n, iter)`: 返回 iter 中 n 个最小的元素

位置 i 的元素总是大于位置 i // 2 的元素（也就是小于位置 2 * i 和 2 * i + 1 的元素）。

代码示例：
```python
from heapq import *
from random import shuffle


data = list(range(10))
shuffle(data)
heap = []
for n in data:
    heappush(heap, n)

print(heap)
heappush(heap, 0.5)
print(heap)
```