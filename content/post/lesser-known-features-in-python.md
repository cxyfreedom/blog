---
title: Python 中容易出现误解的特性
date: 2019-08-30T18:38:05+08:00
lastmod: 2019-08-30T18:38:05+08:00
draft: false
keywords: ["Python", "feature"]
description: ""
tags: ["Python", "feature"]
categories: ["Python"]
---

阅读了[wtfpython](https://github.com/satwikkansal/wtfpython#-be-careful-with-chained-operations)，发现 Python 中还有许多自己不知道的一些细节和特性。本文根据原文记录了一些自己不熟悉和容易忘记的一些点。更多的内容可以直接去看原文。
<!--more-->

## 微妙的字符串

示例：
```python
In [1]: a = "some_string"

In [2]: id(a)
Out[2]: 4467355760

In [3]: id("some" + "_" + "string")
Out[3]: 4467355760

In [4]: a = "wtf"

In [5]: b = "wtf"

In [6]: a is b
Out[6]: True

In [7]: a = "wtf!"

In [8]: b = "wtf!"

In [9]: a is b
Out[9]: False

In [10]: a, b = "wtf!", "wtf!"

In [11]: a is b
Out[11]: False

In [12]: "a" * 20 is "aaaaaaaaaaaaaaaaaaaa"
Out[12]: True

In [13]: "a" * 21 is "aaaaaaaaaaaaaaaaaaaaa"  # 3.7以下版本为False，以上为True
Out[13]: True
```

导致上述结果出现不一致的行为是由于 CPython 在编译优化时，某些情况下会尝试使用已经存在的不可变对象，而不是每次都创建一个新的对象。这种行为被称为**字符串驻留**（string interning）。

发生驻留后，许多变量可能指向内存中的相同字符串对象，从而节省内存。

字符串是隐式驻留的，何时发生隐式驻留取决于具体的实现。有一些方法可以猜测字符串是否会被驻留：

* 所有长度为 0 和长度为 1 的字符串都会被驻留
* 字符串在编译时被实现（`'wtf'` 将被驻留，但是 `''.join(['w', 't', 'f'])` 不会被驻留）
* 字符串中只包含数字、字母或者下划线时将会被驻留，因此 `wtf!` 将不会被驻留，可以在[这里](https://github.com/python/cpython/blob/3.6/Objects/codeobject.c#L19)找到 CPython 中对这部分规则的实现。
* 当在同一行中将 `a` 和 `b` 设置为 257 的时候，Python 解释器会创建一个新对象，然后同时引用第二个变量（**只适用于 3.7 以下版本**，相关讨论参考[这里](https://github.com/satwikkansal/wtfpython/issues/100)）；如果在不同的行上进行赋值操作，就不会知道已经存在一个 257 的对象。
* 常量折叠（constant folding）是 Python 中[窥孔优化](https://en.wikipedia.org/wiki/Peephole_optimization) 技术。这意味着在编译时 `'a'*20` 会被替换成 `aaaaaaaaaaaaaaaaaaaa` 以减少运行时的时钟周期。在 Python3.7 以下版本，只有长度不大于 20 的字符才会发生常量折叠

下面示例分别是 Python3.6 和 Python3.7 中不一致的地方。

Python3.6.4:
```python
>>> a, b = 257, 257
>>> a is b
True
>>> a = 257; b = 257
>>> a is b
True
>>> a, b = int(257), int(257)
>>> a is b
True
>>> a = 257
>>> b = 257
>>> a is b
False
```
Python3.7.3:
```python
>>> a, b = 257, 257
>>> a is b
False
>>> a = 257; b = 257
>>> a is b
True
>>> a, b = int(257), int(257)
>>> a is b
True
>>> a = 257
>>> b = 257
>>> a is b
False
```

Q: 为什么 -5 到 256 这个范围内返回的对象的地址是相同的？

A: 在 Python 的 `main.c` 中，会调用 `_Py_InitializeCore` 来初始化 Python 中的各种模块，源码参考[这里](https://github.com/python/cpython/blob/37fd9f73e2fa439554977cfba427bf94c1fedb6b/Modules/main.c#L3001)。在初始化过程中，`_PyLong_Init` 会被调用，源码实现[位置](https://github.com/python/cpython/blob/37fd9f73e2fa439554977cfba427bf94c1fedb6b/Python/pylifecycle.c#L741)。
`_PyLong_Init` 的作用是初始化 `small_ints` 数组，源码参考[这里](https://github.com/python/cpython/blob/37fd9f73e2fa439554977cfba427bf94c1fedb6b/Objects/longobject.c#L5463)：
```c
int
_PyLong_Init(void)
{
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    int ival, size;
    PyLongObject *v = small_ints;

    for (ival = -NSMALLNEGINTS; ival <  NSMALLPOSINTS; ival++, v++) {
        size = (ival < 0) ? -1 : ((ival == 0) ? 0 : 1);
        if (Py_TYPE(v) == &PyLong_Type) {
            /* The element is already initialized, most likely
             * the Python interpreter was initialized before.
             */
            Py_ssize_t refcnt;
            PyObject* op = (PyObject*)v;

            refcnt = Py_REFCNT(op) < 0 ? 0 : Py_REFCNT(op);
            _Py_NewReference(op);
            /* _Py_NewReference sets the ref count to 1 but
             * the ref count might be larger. Set the refcnt
             * to the original refcnt + 1 */
            Py_REFCNT(op) = refcnt + 1;
            assert(Py_SIZE(op) == size);
            assert(v->ob_digit[0] == (digit)abs(ival));
        }
        else {
            (void)PyObject_INIT(v, &PyLong_Type);
        }
        Py_SIZE(v) = size;
        v->ob_digit[0] = (digit)abs(ival);
    }
#endif
    _PyLong_Zero = PyLong_FromLong(0);
    if (_PyLong_Zero == NULL)
        return 0;
    _PyLong_One = PyLong_FromLong(1);
    if (_PyLong_One == NULL)
        return 0;

    /* initialize int_info */
    if (Int_InfoType.tp_name == NULL) {
        if (PyStructSequence_InitType2(&Int_InfoType, &int_info_desc) < 0)
            return 0;
    }

    return 1;
}
```
当我们创建 `a = 100` 这样的语句时，其实底层会调用 `PyLong_FromLong` 这个函数，源码可以参考[这里](https://github.com/python/cpython/blob/37fd9f73e2fa439554977cfba427bf94c1fedb6b/Objects/longobject.c#L243)：
```c
PyObject *
PyLong_FromLong(long ival)
{
    PyLongObject *v;
    unsigned long abs_ival;
    unsigned long t;  /* unsigned so >> doesn't propagate sign bit */
    int ndigits = 0;
    int sign;

    CHECK_SMALL_INT(ival);

    if (ival < 0) {
        /* negate: can't write this as abs_ival = -ival since that
           invokes undefined behaviour when ival is LONG_MIN */
        abs_ival = 0U-(unsigned long)ival;
        sign = -1;
    }
    else {
        abs_ival = (unsigned long)ival;
        sign = ival == 0 ? 0 : 1;
    }

    /* Fast path for single-digit ints */
    if (!(abs_ival >> PyLong_SHIFT)) {
        v = _PyLong_New(1);
        if (v) {
            Py_SIZE(v) = sign;
            v->ob_digit[0] = Py_SAFE_DOWNCAST(
                abs_ival, unsigned long, digit);
        }
        return (PyObject*)v;
    }

#if PyLong_SHIFT==15
    /* 2 digits */
    if (!(abs_ival >> 2*PyLong_SHIFT)) {
        v = _PyLong_New(2);
        if (v) {
            Py_SIZE(v) = 2*sign;
            v->ob_digit[0] = Py_SAFE_DOWNCAST(
                abs_ival & PyLong_MASK, unsigned long, digit);
            v->ob_digit[1] = Py_SAFE_DOWNCAST(
                  abs_ival >> PyLong_SHIFT, unsigned long, digit);
        }
        return (PyObject*)v;
    }
#endif

    /* Larger numbers: loop to determine number of digits */
    t = abs_ival;
    while (t) {
        ++ndigits;
        t >>= PyLong_SHIFT;
    }
    v = _PyLong_New(ndigits);
    if (v != NULL) {
        digit *p = v->ob_digit;
        Py_SIZE(v) = ndigits*sign;
        t = abs_ival;
        while (t) {
            *p++ = Py_SAFE_DOWNCAST(
                t & PyLong_MASK, unsigned long, digit);
            t >>= PyLong_SHIFT;
        }
    }
    return (PyObject *)v;
}
```

## 相同的哈希

```python
>>> some_dict={}
>>> some_dict[5.5]="Ruby"
>>> some_dict[5.0]="JavaScript"
>>> some_dict[5]="Python"
>>> some_dict[5.5]
'Ruby'
>>> some_dict[5.0]
'Python'
>>> some_dict[5]
'Python'
```
说明：

* Python 的字典通过检查键值是否相等以及比较哈希值来确定两个键是否相同。
* 具有相同值的不可变对象在 Python 中始终具有相同的哈希值。

```python
>>> 5 == 5.0
True
>>> hash(5) == hash(5.0)
True
```
Stackoverflow 上的一个[回答](https://stackoverflow.com/questions/32209155/why-can-a-floating-point-dictionary-key-overwrite-an-integer-key-with-the-same-v/32211042#32211042)很好的解释了这个背后的原理。

## 相同的对象

```python
class WTF:
    def __init__(self):
        print('I')

    def __del__(self):
        print('D')


print(WTF() == WTF())
print(WTF() is WTF())
print(hash(WTF()) == hash(WTF()))
print(id(WTF()) == id(WTF()))
```
输出如下：
```python
I
I
D
D
False
I
I
D
D
False
I
D
I
D
True
I
D
I
D
True
```
当调用 `id` 函数时，Python 创建了一个 `WTF` 对象并传给 `id` 函数，获取其 id 值（内存地址）后，丢弃该对象，该对象也随即被销毁了。当我们连续两次进行这个操作的时候，Python 会将相同的内存地址分配给第二个对象，因此这两个对象的 id 值时相同的。

所以我们可以知道，对象的 id 值仅仅在对象的生命周期内唯一。**在对象被销毁之后或者被创建之前，其他对象可以具有相同的 id 值**。这也就是为什么我们的输出结果不同，**对象销毁的顺序是造成所有不同之处的原因**。

## 执行实际差异

```python
>>> array = [1, 8, 15]
>>> g = (x for x in array if array.count(x) > 0)
>>> array = [2, 8 ,22]
>>> list(g)
[8]
>>> array_1 = [1, 2, 3, 4]
>>> g1 = (x for x in array_1)
>>> array_1 = [1, 2, 3, 4, 5]
>>> list(g1)
[1, 2, 3, 4]
>>> array_2 = [1, 2, 3, 4]
>>> g2 = (x for x in array_2)
>>> array_2[:] = [1, 2, 3, 4, 5]
>>> list(g2)
[1, 2, 3, 4, 5]
```
解释：

结论一：在生成器表达式中，`in` 子句是在声明时执行，条件子句是在运行时才执行。

因此第一个示例中，`array` 已经被重新赋值为 `[2, 8, 22]`，因此对于之前的数值，只有 `count(8)` 的结果是大于 0 的，所以生成器只会生成 8。

第二个示例中，`g1` 和 `g2` 的差异在于被重新赋值的方式不同导致的。

第一种情况下，`array_1` 虽然被绑定到了新的对象上，但是由于 `in` 子句是在声明时执行的，因此引用的还是旧的对象。

第二种情况，`array_2` 的切片操作是将原来的对象原地更新为新的对象，而 `g2` 和 `array_2` 引用的还是同一个对象，因此更新为了 `[1, 2, 3, 4, 5]`。

## 闭包

```python
funcs = []
results = []
for x in range(7):
    def some_func():
        return x
    funcs.append(some_func)
    results.append(some_func())

funcs_results = [func() for func in funcs]
print(funcs_results)  # [6, 6, 6, 6, 6, 6, 6]

powers_of_x = [lambda x: x**i for i in range(10)]
print([f(2) for f in powers_of_x])  # [512, 512, 512, 512, 512, 512, 512, 512, 512, 512] 
```
解释：

* 当在循环内部定义函数时，如果该函数的主体中使用了循环变量，那么闭包函数将和循环变量进行绑定，而不是它的值。因此，所有的函数都会使用最后分配给循环变量的值来进行计算。
* 为了避免这种情况，可以将循环变量作为参数传递给函数来获得预期的效果。因为这个时候会在函数内定义一个局部变量。

```python
funcs = []
results = []
for x in range(7):
    def some_func(x=x):
        return x
    funcs.append(some_func)
    results.append(some_func())

funcs_results = [func() for func in funcs]
print(funcs_results)  # [0, 1, 2, 3, 4, 5, 6]
```

## 类属性和实例属性

```python
class A:
    x = 1


class B(A):
    pass


class C(A):
    pass


print(A.x, B.x, C.x)  # 1 1 1
B.x = 2
print(A.x, B.x, C.x)  # 1 2 1
A.x = 3
print(A.x, B.x, C.x)  # 3 2 3
a = A()
print(a.x, A.x)  # 3 3
a.x += 1
print(a.x, A.x)  # 4 3


class SomeClass:
    some_var = 15
    some_list = [5]
    another_list = [5]

    def __init__(self, x):
        self.some_var = x + 1
        self.some_list = self.some_list + [x]
        self.another_list += [x]


some_obj = SomeClass(100)
print(some_obj.some_list)  # [5, 100]
print(some_obj.another_list)  # [5, 100]
another_obj = SomeClass(200)
print(another_obj.some_list)  # [5, 200]
print(another_obj.another_list)  # [5, 100, 200]
print(another_obj.another_list is SomeClass.another_list)  # True
print(another_obj.another_list is some_obj.another_list)  # True
```
解释：

* 类变量和实例变量在内部是通过类对象的字典来处理。如果在当前类的字典中找不到的话就去父类中寻找。
* `+=` 运算符会原地修改可变对象，而不是创建新的对象。因此，在这种情况下，修改一个实例的属性会影响到其他实例和类属性。

## yield 生成 None

```python
some_iterable = ('a', 'b')


def some_func(val):
    return "something"


print([x for x in some_iterable])  # ['a', 'b']
print([(yield x) for x in some_iterable])  # <generator object <listcomp> at 0x1155af620>
print(list([(yield x) for x in some_iterable]))  # ['a', 'b']
print(list((yield x) for x in some_iterable))  # ['a', None, 'b', None]
print(list(some_func((yield x)) for x in some_iterable))  # ['a', 'something', 'b', 'something']
```
解释：

> 在 Cpython 的推导式和生成器表达式中处理 `yield` 是一个 BUG，在 Python3.8 中已经修复。在 Python3.7 中作为一个废弃的警告。

如果你尝试使用 `dis` 模块来反汇编分析一个生成器表达式：
```python
>>> dis.dis(compile("(i for i in range(3))", '', 'exec'))
  1           0 LOAD_CONST               0 (<code object <genexpr> at 0x108f9dd20, file "", line 1>)
              2 LOAD_CONST               1 ('<genexpr>')
              4 MAKE_FUNCTION            0
              6 LOAD_NAME                0 (range)
              8 LOAD_CONST               2 (3)
             10 CALL_FUNCTION            1
             12 GET_ITER
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 LOAD_CONST               3 (None)
             20 RETURN_VALUE

Disassembly of <code object <genexpr> at 0x108f9dd20, file "", line 1>:
  1           0 LOAD_FAST                0 (.0)
        >>    2 FOR_ITER                10 (to 14)
              4 STORE_FAST               1 (i)
              6 LOAD_FAST                1 (i)
              8 YIELD_VALUE
             10 POP_TOP
             12 JUMP_ABSOLUTE            2
        >>   14 LOAD_CONST               0 (None)
             16 RETURN_VALUE
```
从上述的字节码可以看出，`yield` 表达式在上下文中工作，因为编译器将这些看成是伪装的函数（`functions-in-disguise`）。

但是这是一个 BUG，在 Python3.7 之前的语法中允许，但是 `yield` 表达式描述中说明它不能在比如生成器表达式、列表推导式中使用。

> The yield expression is only used when defining a generator function and thus can only be used in the body of a function definition.

在 Python3.8 中，在推导式中使用 `yield` 或者 `yield from` 会抛出语法错误的异常，在 Python3.7 中会抛出 `DeprecationWarning`，提示不要在代码中使用这种方式。
```python
# Python3.7
>>> import warnings
>>> warnings.simplefilter('error')
>>> [(yield i) for i in range(3)]
  File "<stdin>", line 1
SyntaxError: 'yield' inside list comprehension
>>> warnings.simplefilter('always')
>>> [(yield i) for i in range(3)]
<stdin>:1: DeprecationWarning: 'yield' inside list comprehension
<generator object <listcomp> at 0x10950f390>
```
在列表推导式中使用 `yield` 和在生成器表达式中使用 `yield`，这两者的区别在于实现方式。列表推导式使用 `LIST_APPEND` 将堆栈顶部的内容放到列表中，而生成器表达式则是 `yield` 这个值。
```python
>>> dis.dis(compile("[(yield i) for i in range(3)]", '', 'exec'))
  1           0 LOAD_CONST               0 (<code object <listcomp> at 0x108f9dd20, file "", line 1>)
              2 LOAD_CONST               1 ('<listcomp>')
              4 MAKE_FUNCTION            0
              6 LOAD_NAME                0 (range)
              8 LOAD_CONST               2 (3)
             10 CALL_FUNCTION            1
             12 GET_ITER
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 LOAD_CONST               3 (None)
             20 RETURN_VALUE

Disassembly of <code object <listcomp> at 0x108f9dd20, file "", line 1>:
  1           0 BUILD_LIST               0
              2 LOAD_FAST                0 (.0)
        >>    4 FOR_ITER                10 (to 16)
              6 STORE_FAST               1 (i)
              8 LOAD_FAST                1 (i)
             10 YIELD_VALUE
             12 LIST_APPEND              2
             14 JUMP_ABSOLUTE            4
        >>   16 RETURN_VALUE
>>> dis.dis(compile("((yield i) for i in range(3))", '', 'exec'))
  1           0 LOAD_CONST               0 (<code object <genexpr> at 0x108f9dd20, file "", line 1>)
              2 LOAD_CONST               1 ('<genexpr>')
              4 MAKE_FUNCTION            0
              6 LOAD_NAME                0 (range)
              8 LOAD_CONST               2 (3)
             10 CALL_FUNCTION            1
             12 GET_ITER
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 LOAD_CONST               3 (None)
             20 RETURN_VALUE

Disassembly of <code object <genexpr> at 0x108f9dd20, file "", line 1>:
  1           0 LOAD_FAST                0 (.0)
        >>    2 FOR_ITER                12 (to 16)
              4 STORE_FAST               1 (i)
              6 LOAD_FAST                1 (i)
              8 YIELD_VALUE
             10 YIELD_VALUE
             12 POP_TOP
             14 JUMP_ABSOLUTE            2
        >>   16 LOAD_CONST               0 (None)
             18 RETURN_VALUE
```
对于列表推导式，每次 `yield` 一个值放到栈顶。而生成器表达式则是 `yield` 值放到栈顶部后再进行一次 `yield`，此时栈中包含了 `yield` 生成的值和第二次 `yield` 出的 `None`。

列表推导式能够正确的返回我们预期的结果，这是因为在 Python3 中将异常值附加到了 `StopInteration` 异常中。
```python
>>> from itertools import islice
>>> listgen = [(yield i) for i in range(3)]
<stdin>:1: DeprecationWarning: 'yield' inside list comprehension
>>> list(islice(listgen, 3))
[0, 1, 2]
>>> try:
...     next(listgen)
... except StopIteration as si:
...     print(si.value)
...
[None, None, None]
```
这些 `None` 值就是从 `yield` 表达式的返回值。这个问题也同样存在字典推导式和集合推导式中。
```python
# Python2.7
>>> list({(yield k): (yield v) for k, v in {'foo': 'bar', 'spam': 'eggs'}.items()})
['bar', 'foo', 'eggs', 'spam', {None: None}]
>>> list({(yield i) for i in range(3)})
[0, 1, 2, set([None])]
dis.dis(compile("list({(yield k): (yield v) for k, v in {'foo': 'bar', 'spam': 'eggs'}.items()})", '', 'exec'))
```

相关参考：

* [issue 10544](http://bugs.python.org/issue10544)
* [yield in list comprehensions and generator expressions](https://stackoverflow.com/questions/32139885/yield-in-list-comprehensions-and-generator-expressions)
* [dis](https://www.osgeo.cn/cpython/library/dis.html)

## 可变对象/不可变对象

```python
>>> some_tuple = ("A", "tuple", "with", "values")
>>> another_tuple = ([1, 2], [3, 4], [5, 6])
>>> some_tuple[2] = "change this"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> another_tuple[2].append(1000)
>>> another_tuple
([1, 2], [3, 4], [5, 6, 1000])
>>> another_tuple[2] += [99, 999]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> another_tuple
([1, 2], [3, 4], [5, 6, 1000, 99, 999])
```
解释：

对于 `+=` 这个运算符，例如 `a += b`：

* 对于可变对象，操作结果会直接在 `a` 变量上原地修改，`a` 所对应的地址不变。
* 对于不可变对象，`+=` 则是等价于 `a = a + b`，会产生新的变量，然后绑定到 `a` 上。

```python
>>> a = [1, 2, 3]
>>> id(a)
4432216584
>>> a += [4, 5]
>>> a
[1, 2, 3, 4, 5]
>>> id(a)
4432216584
>>> b = (1, 2, 3)
>>> id(b)
4432190920
>>> b += (4, 5)
>>> id(b)
4432178256
```
下面再来看几种情况：
```python
>>> t = (1, 2, [3, 4])
>>> t[2] = [5, 6]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>>
>>> t = (1, 2, [3, 4])
>>> t[2].extend([5, 6])
>>> t
(1, 2, [3, 4, 5, 6])
>>> t[2].append(7)
>>> t
(1, 2, [3, 4, 5, 6, 7])
```
上面第一种情况出错是因为 `=` 操作产生了 `assign` 的操作，而 `tuple` 中的元素不支持赋值操作。

第二种情况是因为 `extend/append` 是修改了列表的元素，但是列表本身的 `id` 并没有变化。

我们使用 `dis` 模块来分析一下 `+=` 和 `extend` 的区别。
```python
t = (1,2, [30,40])
t[2] += [50,60]
t[2].extend([70, 80])
```
对于上述代码执行 `python -m dis main.py`，输出如下：
```python
...
  7          18 LOAD_NAME                1 (t)
             20 LOAD_CONST               2 (2)
             22 DUP_TOP_TWO
             24 BINARY_SUBSCR
             26 LOAD_CONST               5 (50)
             28 LOAD_CONST               6 (60)
             30 BUILD_LIST               2
             32 INPLACE_ADD
             34 ROT_THREE
             36 STORE_SUBSCR

  8          38 LOAD_NAME                1 (t)
             40 LOAD_CONST               2 (2)
             42 BINARY_SUBSCR
             44 LOAD_METHOD              2 (extend)
             46 LOAD_CONST               7 (70)
             48 LOAD_CONST               8 (80)
             50 BUILD_LIST               2
             52 CALL_METHOD              1
             54 POP_TOP
             56 LOAD_CONST               9 (None)
```

* `24 BINARY_SUBSCR` 表示将 `t[2]` 的值放在栈的顶部；
* `32 INPLACE_ADD` 表示 `TOS += [50, 60]`，这一步执行是成功的；
* `42 STORE_SUBSCR` 表示 `t[2] = TOS`，这里产生了一个赋值操作，但是 `tuple` 中的元素是不支持的，因此会抛出异常，但是此时列表的修改已经完成。

对于 `tuple`，`+=` 并不是原子操作。而是 `extend` 和 `=` 两个操作步骤。

参考：

* [Python中tuple+=赋值的四个问题](https://segmentfault.com/a/1190000010767068)

## 消失的外部变量

```python
e = 7
try:
    raise Exception()
except Exception as e:
    pass

print(e)
```
执行上述代码，输出：
```python
NameError: name 'e' is not defined
```
当使用 `as` 将目标赋值为一个异常时，它将在 except 子句结束时被清除。这就相当于：
```python
except E as D:
    foo
```
被翻译成为：
```python
except E as N:
    try:
        foo
    finally:
        del N
```
这意味着异常必须赋值给一个不同的名称才能在 except 子句之后引用它。 异常会被清除是因为在附加了回溯信息的情况下，它们会形成堆栈帧的循环引用，使得所有局部变量保持存活直到发生下一次垃圾回收。

参考：

* [try 语句](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#except)

## 子类关系

```python
>>> from collections.abc import Hashable
>>> issubclass(list, object)
True
>>> issubclass(object, Hashable)
True
>>> issubclass(list, Hashable)
False
```
* Python 中的子类关系并不一定是可传递的，只要在元类中定义 `__subclasscheck__` 方法即可。
* 当 `issubclass(cls, Hashable)` 被调用时，它只是在 `cls` 中寻找 `__hash__` 方法或者从继承的父类中寻找。

```python
class MyMetaClass(type):
    def __subclasscheck__(cls, subclass):
        print("Whateva, I do what I want!")
        import random
        return random.choice([True, False])


class MyClass(metaclass=MyMetaClass):
    pass


print(issubclass(list, MyClass))
# output
# Whateva, I do what I want!
# False or True
```
详细解释可以参考另一篇[文章](https://www.naftaliharris.com/blog/python-subclass-intransitivity/)。

## 键型转换

```python
>>> class SomeClass(str):
...     pass
...
>>> some_dict = {'s': 42}
>>> type(list(some_dict.keys())[0])
<class 'str'>
>>> s = SomeClass('s')
>>> some_dict[s] = 40
>>> some_dict
{'s': 40}
>>> type(list(some_dict.keys())[0])
<class 'str'>
```
解释：

由于 `SomeClass` 会从 `str` 中自动继承 `__hash__` 方法，因此 `s` 对象和 `"s"` 字符串的哈希值是想用的。`SomeClass('s') == 's'` 是因为 `SomeClass` 也继承了 `str` 中的 `__eq__` 方法。 

为了让这两者是不同的键，我们可以重新定义 `SomeClass` 的 `__eq__` 方法。
```python
class SomeClass(str):
    def __eq__(self, other):
        return (
                type(self) is SomeClass
                and type(other) is SomeClass
                and super().__eq__(other)
        )

    # 当我们自定义 __eq__ 方法时, Python 不会再自动继承 __hash__ 方法
    # F所以我们也需要定义它
    __hash__ = str.__hash__


some_dict = {'s': 42}
s = SomeClass('s')
some_dict[s] = 40
print(some_dict)  # {'s': 42, 's': 40}
keys = list(some_dict.keys())
print(type(keys[0]), type(keys[1]))  # <class 'str'> <class '__main__.SomeClass'>
```

## 赋值

```python
>>> a, b = a[b] = {}, 5
>>> a
{5: ({...}, 5)}
```
解释：

在 Python 中，赋值语句的形式如下：
```python
(target_list "=") + (expression_list | yield_expression)
```

> 赋值语句计算表达式列表并将单个结果对象从左到右依次分配给目标列表中的每一项。

赋值语句中的 `+` 意味着可以有一个或者多个目标列表，在示例中，`a, b` 和 `a[b]` 是目标列表，`{}, 5` 是表达式列表。（**表达式列表只能存在一个**）。

表达式列表计算结束后，将自动解包从左到右分配给目标列表。在示例中，首先将 `{}, 5` 分别赋值给 `a` 和 `b`，然后得到 `a = {}` 和 `b = 5`。

然后将表达式列表的值赋给第二个目标列表 `a[b]`，键为 5，值设置为元组 `({}， 5)` 来创建循环引用（输出中的 `{...}` 和 `a` 引用了相同的对象）

上述赋值等同于：
```python
a, b = {}, 5
a[b] = a, b
a[b][0] is a  # True
```

## 空间移动

```python
import numpy as np

def energy_send(x):
    # 初始化一个 numpy 数组
    np.array([float(x)])

def energy_receive():
    # 返回一个空的 numpy 数组
    return np.empty((), dtype=np.float).tolist()


energy_send(123.456)
energy_receive()  # output: 123.456
```
解释：

在 `energy_send` 函数中创建的 `numpy` 数组并没有返回，因此内存空间被释放并且可以被重新分配。

对于 `np.empty()` 函数来说，它直接返回下一段空闲内存，而不重新进行初始化。这个内存点刚好是刚刚释放的那部分内存（**通常情况下，但是并不是绝对**）

## 迭代字典的修改

```python
x = {0: None}

for i in x:
    del x[i]
    x[i+1] = None
    print(i)
```
在 Python3.7 版本执行上述代码，输出如下：
```python
0
1
2
3
4
```
解释：

这并不是 BUG，在[文档](https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects)中写着这么一段话：
```
Iterating views while adding or deleting entries in the dictionary may raise a RuntimeError or fail to iterate over all entries.
```
之所以只运行 5 次，是因为在目前的版本实现中字典[初始化的容量是 8](https://github.com/python/cpython/blob/69c0db5050f623e8895b72dfe970392b1f9a0e2e/Objects/dictobject.c#L103)，当容量超过 `2/3` 的时候，即 `8 * 2/3 == 5.33333`，会触发扩容并中断当前迭代。当你删除键的时候，就会填充 [DKIX_DUMMY](https://github.com/python/cpython/blob/v3.6.1/Objects/dictobject.c#L76-L81),

参考：

* [Modifying a dictionary while iterating over it. Bug in Python dict?](https://stackoverflow.com/questions/44763802/modifying-a-dictionary-while-iterating-over-it-bug-in-python-dict)

## del 删除

示例一
```python
>>> class SomeClass:
...     def __del__(self):
...         print("Deleted!")
...
>>> x = SomeClass()
>>> y = x
>>> del x
>>> del y
Deleted!
```
示例二
```python
>>> class SomeClass:
...     def __del__(self):
...         print("Deleted!")
...
>>> x = SomeClass()
>>> y = x
>>> del x
>>> y
<__main__.SomeClass object at 0x10db7df60>
>>> del y
>>> _
<__main__.SomeClass object at 0x10db7df60>
>>> globals()
Deleted!
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'SomeClass': <class '__main__.SomeClass'>}
```
解释：

`del x` 并不会立即调用 `x.__del__()`。每当遇到 `del x` 语句时，Python 回将 `x` 的引用计数减一，当引用计数为零时，就会调用 `x.__del__()`。

在第二个示例中，第一次执行 `del y` 没有执行对应的 `__del__` 方法是因为之前的 `y` 创建了同一个对象的另一个引用，因此此时引用计数为 1。

`_` 会自动保存上一个表达式输出非 `None` 的值。当有新的表达式输出非 `None` 的值，`_` 就会保存新的输出值。

执行 `globals()` 后，会导致之前的引用被销毁，此时 `SomeClass` 的引用计数就变为了零，就会调用 `__del__` 来回收对象。

## 迭代列表时删除元素

```python
list_1 = [1, 2, 3, 4]
list_2 = [1, 2, 3, 4]
list_3 = [1, 2, 3, 4]
list_4 = [1, 2, 3, 4]

for idx, item in enumerate(list_1):
    del item

for idx, item in enumerate(list_2):
    list_2.remove(item)

for idx, item in enumerate(list_3[:]):
    list_3.remove(item)

for idx, item in enumerate(list_4):
    list_4.pop(idx)


print(list_1)  # [1, 2, 3, 4]
print(list_2)  # [2, 4]
print(list_3)  # []
print(list_4)  # [2, 4]
```
解释：

关于 `del`、`remove`、`pop` 三者的区别：

* `del var_name` 只是从本地或者全局的命名空间删除了 `var_name` 变量
* `remove` 会删除第一个匹配到的指定值，而不是索引，如果找不到值则抛出 `ValueError` 异常
* `pop` 会删除指定索引位置的元素并返回它，如果索引不合法则抛出 `IndexError` 异常

`list_2` 和 `list_4` 输出结果是 `[2, 4]` 的原因是当删除 `1` 的时候，列表的实际内容就变成了 `[2, 3, 4]`，元素的索引也相应发生了变化，下一次迭代时，索引为 1 的元素就变成了 `3`，所以删除的元素就是 `3`。

通过自定义迭代器来查看这个流程：
```python
class CustomIterator(object):
    def __init__(self, seq):
        self.seq = seq
        self.idx = 0

    def __iter__(self):
        return self

    def __next__(self):
        print('give next element:', self.idx)
        for idx, item in enumerate(self.seq):
            if idx == self.idx:
                print(idx, '--->', item)
            else:
                print(idx, '    ', item)
        try:
            nxtitem = self.seq[self.idx]
        except IndexError:
            raise StopIteration
        self.idx += 1
        return nxtitem

    next = __next__  # py2 compat


some_list = [1, 2, 3, 4]

for idx, item in enumerate(CustomIterator(some_list)):
    del some_list[idx]
```
执行上述代码输出：
```python
give next element: 0
0 ---> 1
1      2
2      3
3      4
give next element: 1
0      2
1 ---> 3
2      4
give next element: 2
0      2
1      4
```
相似的还有在字典中修改键的操作（说明：实际开发中，切不要再迭代字典的同时修改字典内容），具体的例子和原因可以参考下面提供的链接。

参考：

* [What happens when you try to delete a list element while iterating over it](https://stackoverflow.com/questions/45946228/what-happens-when-you-try-to-delete-a-list-element-while-iterating-over-it)
* [How to change all the dictionary keys in a for loop with d.items()?](https://stackoverflow.com/questions/45877614/how-to-change-all-the-dictionary-keys-in-a-for-loop-with-d-items)

## 循环变量泄漏

示例一：
```python
for x in range(7):
    if x == 6:
        print(x, ': for x inside loop')
print(x, ': x in global')
```
输出：
```python
6 : for x inside loop
6 : x in global
```
示例二：
```python
x = -1
for x in range(7):
    if x == 6:
        print(x, ': for x inside loop')
print(x, ': x in global')
```
输出：
```python
6 : for x inside loop
6 : x in global
```
示例三：
```python
x = 1
print([x for x in range(5)])
print(x, ': x in global')
```
在 Python2 下输出：
```python
[0, 1, 2, 3, 4]
(4, ': x in global')
```
在 Python3 下输出：
```python
[0, 1, 2, 3, 4]
(1, ': x in global')
```
解释：

* 在 Python 中，`for` 循环所在作用域并在结束后保留定义的循环变量。如果我们在全局命名空间中定义过该循环变量，那么会重新绑定现有的变量。
* 在列表推导式中，Python3 的循环变量不会再泄漏到周围的作用域中。

## 链式操作的陷阱

示例：
```python
>>> (False == False) in [False]
False
>>> False == (False in [False])
False
>>> False == False in [False]
True
>>> True is False == False
False
>>> False is False is False
True
>>> 1 > 0 < 1
True
>>> (1 > 0) < 1
False
>>> 1 > (0 < 1)
```
解释：

在[文档](https://docs.python.org/3/reference/expressions.html#comparisons)中，有这么一段话：

> Formally, if a, b, c, …, y, z are expressions and op1, op2, …, opN are comparison operators, then a op1 b op2 c ... y opN z is equivalent to a op1 b and b op2 c and ... y opN z, except that each expression is evaluated at most once.

根据描述，`False is False is False` 相当于 `(False is False) and (False is False)`，`True is False == False` 则相当于 `(True is False) and (False == False)`，其他示例也是如此。

## 内置彩蛋

`antigravity` 是 Python 开发人员发布的少数复活节彩蛋之一。执行 `import antigravity` 会打开一个 Python 的 XKCD 漫画页面。

