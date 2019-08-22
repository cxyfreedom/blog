---
title: 深入理解 Python 中的元类（metaclass）
date: 2019-04-24T17:03:19+08:00
description: ""
tags: ["Python", "metaclass", "元类"]
categories:
---
元类（metaclass）是 Python 中一个比较难理解的概念，找了许多资料和官方文档，花了好久才算基本理解清楚。本文记录了多篇文章中的内容和自己的一些理解，便于自己和他人查漏补缺。

<!--more-->
## 类也是对象

只要使用关键字 `class`，Python 解释器就会在执行的时候创建一个对象。类的本质也是一个对象，可以对其作如下操作：

* 赋值给一个变量
* 拷贝它
* 增加一个属性
* 将它作为函数参数进行传递

```python
In [1]: class ObjectCreator(object):
   ...:     pass
   ...:

In [2]: print(ObjectCreator)
<class '__main__.ObjectCreator'>

In [3]: def echo(o):
   ...:     print(o)
   ...:

In [4]: echo(ObjectCreator)
<class '__main__.ObjectCreator'>

In [5]: hasattr(ObjectCreator, 'new_attribute')
Out[5]: False

In [6]: ObjectCreator.new_attribute = 'foo'

In [7]: ObjectCreator.new_attribute
Out[7]: 'foo'

In [8]: ObjectCreatorMirror = ObjectCreator

In [9]: ObjectCreatorMirror
Out[9]: __main__.ObjectCreator
```

## 动态地创建类

type 可以动态创建类。接受一个类的描述作为参数，然后返回一个类。

```
type(类名, 父类的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)
```

手动创建例子：
```python
>>> MyShinyClass = type('MyShinyClass', (), {})
>>> MyShinyClass
<class '__main__.MyShinyClass'>
>>> MyShinyClass()
<__main__.MyShinyClass object at 0x10bfcb358>
```

接受一个字典来为类定义属性：
```python
class Foo:
    bar = True

# 等同于
Foo = type('Foo', (), {'bar':True})

# 继承
class FooChild(Foo):
    pass

FooChild = type('FooChild', (Foo,),{})
FooChild.bar # True
```

如果希望为类增加一个方法，只需要创建一个函数并将其作为属性赋值给类对象即可：
```python
def echo_bar(self):
    print(self.bar)

FooChild = type('FooChild', (Foo, ), {'echo_bar': echo_bar})
hasattr(Foo, 'echo_bar')  # False
hasattr(FooChild, 'echo_bar')  # True
my_foo = FooChild()
my_foo.echo_bar()  # True
```

<p id="div-border-left-green">说明：使用 type 创建的类和使用元类的类，都是新式类。</p>

### type 和 `type.__new__` 的区别

为了说明这个问题，我们直接从代码来看：
```python
class MetaA(type):
    def __new__(cls, name, bases, dct):
        print('MetaA.__new__')
        return type(name, bases, dct)

    def __init__(self, name, bases, dct):
        print('MetaA.__init__')


class A(metaclass=MetaA):
    pass

# output: MetaA.__new__


class MetaB(type):
    def __new__(cls, name, bases, dct):
        print('MetaB.__new__')
        return type.__new__(cls, name, bases, dct)

    def __init__(self, name, bases, dct):
        print('MetaB.__init__')


class B(metaclass=MetaB):
    pass

# output: 
# MetaB.__new__
# MetaB.__init__
```

从代码中可以看出，在 MetaA 中通过 `__new__` 方法返回了一个新的类，但是这个类和 MetaA 没有任何关系，因此，MetaA 的 `__init__` 也就不会调用，因为 `__init__` 是用来控制 MetaA 实例的初始化的。

而在 MetaB 中通过 `__new__` 方法返回了一个 MetaB 的实例对象 B，B 是一个类，因此下一步就会调用 `__init__` 来初始化这个实例对象。

官方文档说明：
<p id="div-border-left-green">If __new__() returns an instance of cls, then the new instance’s __init__() method will be invoked like __init__(self[, ...]), where self is the new instance and the remaining arguments are the same as were passed to __new__().

If __new__() does not return an instance of cls, then the new instance’s __init__() method will not be invoked.</p>

### type 和 object 的关系

在 Python 中，查看一个类型的父类，可以通过 `__bases__` 属性来查看。要查看一个实例的类型，可以通过 `__class__` 属性查看，或者使用 `type()` 函数查看。

type 和 object 都是 type 的实例。在 Python 中，object 是父子关系的顶端，也就是说所有数据类型的父类都是 object；type 是类型实例关系的顶端，也就是说所有对象都是 type 的实例。
```python
type(object)  # type
type(type)    # type
# object 是 type 的一个实例
object.__class__  # type
# type 也是 type 的一个实例
type.__class__  # type
object.__bases__ # ()
# type 是 object 的子类
type.__bases__  # (object,)
```

用一张图概括：

{{% figure class="center" src="/img/metaclass.jpg" alt="metaclass" %}}

白板虚线表示源是目标的实例，实线表示源是目标的子类。即，左边的是右边的类型，而上面的是下面的父亲。

虚线是跨列产生关系，而实线只能在一列内产生关系。除了 type 和 object。

图中的第一列，我们将其称为 Type。
图中的第二列，既是第三列的类型，也是第一列的实例，我们将其称为 TypeObject。
图中的第三列，是第二列的实例，没有父类，即没有 `__bases__` 属性，我们将其称为 Instance。

我们希望在第一列添加新的东西，那么必须是 type 的子类：
```python
class M(type):
    pass

M.__class__  # type
M.__bases__  # type
```

M 的类型和父类都是 type，因此 M 属于第一列，M 也就是我们所说的**元类**！type 也是所有元类的父类。

如果我们想要实例化元类，我们还需要定义一个类：
```python
# python3 方式
class TM(metaclass=M)
    pass

M.__class__  # M
M.__bases__  # (object,)
```

这个时候 TM 类的类型就不再是 type，而是 M 了，TM 属于第二列。

Q: **type 和 object 同时存在的原因？**

A: 如果 type 和 object 只存在一个，肯定是保留 object。那么这就和大部分的静态语言类型架构类似，但是这样就失去了**动态创建类型**的特性。本来 TypeObject 是一个对象，对象可以在运行时去动态地修改，因此我们定义一个类之后可以去修改它的属性或者行为。如果去掉了 type，那么类就变成了一个纯类型，运行的时候就无法变动。

## 元类（metaclass）是什么？

元类就是创造类的类。type 实际上就是一个元类，type 就是 Python 在背后用来创建所有类的元类。

元类在 Python2 和 Python3 中的定义方法不同：
```python
# Python2
class MyClass(object):
    __metaclass__ = MyMeta

# Python3
class MyClass(metaclass=MyMeta):
    pass
```

如果我们希望能够同时兼容 2 和 3，就需要另外使用一种方法。元类有两个基本的特性：

* 元类实例化后得到类
* 元类能够被子类继承

基于这两个特性，我们不难得出解决方案：首先使用元类创建出一个临时类，然后用定义类来继承这个临时类。我们根据这个方案可以写入如下的代码：
```python
# -*- coding: utf-8 -*-
def with_metaclass(meta, *bases):
    """
    :param meta: metaclass
    :param bases: base class
    :return:
    """
    return meta('temp_class', bases, {})


class TestMeta(type):
    def __new__(cls, name, bases, dct):
        return type.__new__(cls, name, bases, dct)


class Foo:
    pass


class Bar(with_metaclass(TestMeta, Foo)):
    pass


print(Bar.mro())  # [<class '__main__.Bar'>, <class '__main__.temp_class'>, <class '__main__.Foo'>, <class 'object'>]
```

我们创建了一个以 TestMeta 为元类，继承 Foo 的 Bar 类。但是在 mro 中混入了一个 `temp_class` 的临时类，我们希望这个最好可以过滤掉。

在 six 模块中，就很好的解决了这个临时类的问题。

### six 模块的 with_metaclass 函数

six 模块是专门解决 Python2 和 Python3 的兼容问题，模块中有一个 `with_metaclass` 函数，代码如下：
```python
def with_metaclass(meta, *bases):
    """Create a base class with a metaclass."""
    # This requires a bit of explanation: the basic idea is to make a dummy
    # metaclass for one level of class instantiation that replaces itself with
    # the actual metaclass.
    class metaclass(type):

        def __new__(cls, name, this_bases, d):
            return meta(name, bases, d)

        @classmethod
        def __prepare__(cls, name, this_bases):
            return meta.__prepare__(name, bases)
    return type.__new__(metaclass, 'temporary_class', (), {})
```

结合我们代码来分析一下这段代码：
```python
class TestMeta(type):
    def __new__(cls, name, bases, dct):
        print(f'{cls} __new__ is called.')
        return type.__new__(cls, name, bases, dct)


class Foo:
    pass


Tmp = with_metaclass(TestMeta, Foo)
# 执行到这里无输出


class Bar(Tmp):
    pass
# 执行到这里输出：
# <class '__main__.with_metaclass.<locals>.metaclass'> __new__ is called.
# <class '__main__.TestMeta'> __new__ is called.
```

执行 `with_metaclass(TestMeta, Foo)` 的时候，内部通过 `type.__new__(metaclass, 'temporary_class', (), {})` 创建了一个临时类，它是 metaclass 类的实例，它的元类是 metaclass，创建的时候仅仅调用了 type 的 `__new__` 方法。

然后定义 Bar 类的时候，Bar 得到了继承的 metaclass，然后调用 metaclass 的 `__new__` 方法实例化，返回 `meta(name, bases, d)`，meta 是 `TestMeta`， bases 是 `(Foo,)`,最后调用 `TestMeta.__new__` 实例化得到 Bar。

### 元类的 `__prepare__` 方法

这里提一下元类中的 `__prepare__` 特殊方法，这个方法是 [PEP 3115](https://www.python.org/dev/peps/pep-3115) 中引入的，作用是用来获取类的命名空间。这个方法只能在元类中使用，而且必须声明为类方法。

参数：第一个参数为元类，第二个参数是要构建类的名称，第三个参数是基类组成的元组，返回值必须是**映射**。

原理：使用元类构建新类时（使用 class 关键字），解释器首先会调用 `__prepare__` 方法，使用类定义体中的属性创建映射，然后将返回的映射传递给 `__new__` 方法的最后一个参数，最后在传递给 `__init__` 方法。

## metaclass 属性

在写一个类的时候，可以为其添加 `__metaclass__` 属性。

当我们创建一个类时，Python 首先会在类的定义中寻找 `__metaclass__` 属性或者查看是否使用 `metaclass=xxx`，如果找到了，Python 就会用它来创建类，如果没有找到，就会在其任何父类中进行寻找 `__class__`，如果都找不到该属性，就会用内建的 `type` 来创建这个类。

Q: 该属性中存放什么代码？
A: 可以创建一个类的东西。type 或者任何使用到 type 或者子类化 type 的东西都可以。

## 自定义元类

`__metaclass__` 并不一定要在类中定义，如果它在模块中定义，那么这个模块中的所有类都会通过这个元类来创建。例如我们希望模块中的类的属性都是大写属性：
```python
# -*- coding: utf-8 -*-
def upper_attr(future_class_name, future_class_parents, future_class_attr):
    """返回一个类对象，将属性都转换为大写"""
    attrs = ((name, value) for name, value in future_class_attr.items() if
             not name.startswith('__'))
    # 将它们转换为大写形式
    uppercase_attr = dict((name.upper(), value) for name, value in attrs)
    # 通过 type 来做类对象的创建
    return type(future_class_name, future_class_parents, uppercase_attr)

# python2 使用 __metaclass__，python3 中使用 metaclass
class Foo(metaclass=upper_attr):
    bar = 'bip'


print(hasattr(Foo, 'bar'))
print(hasattr(Foo, 'BAR'))

f = Foo()
print(f.BAR)
print(Foo.__class__)  # <class 'type'>
```

我们还可以通过真正的 class 来当作元类：
```python
class UpperAttrMetaClass(type):
    def __new__(cls, class_name, class_parents, class_attr):
        attrs = ((name, value) for name, value in class_attr.items() if not name.startswith('__'))
        uppercase_attr = dict((name.upper(), value) for name, value in attrs)
        return super().__new__(cls, class_name, class_parents, uppercase_attr)


class Foo(metaclass=UpperAttrMetaClass):
    bar = 'bip'


print(hasattr(Foo, 'bar'))  # False
print(hasattr(Foo, 'BAR'))  # True

f = Foo()
print(f.BAR)  # bip
print(Foo.__class__)  # <class '__main__.UpperAttrMetaClass'>
```

另外在 Python3 中，如果想要调用关键字参数，例如：
```python
class Foo(object, metaclass=Thing, kwarg1=value1):
    ...
```

那么在 metaclass 中应该写入如下格式：
```python
class Thing(type):
    def __new__(cls, clsname, bases, dct, kwargs1=default):
        ...
```

元类本身并不复杂，它主要负责拦截类的创建、修改类和返回修改之后的类。

## 为什么要使用元类？

> “元类就是深度的魔法，99%的用户应该根本不必为此操心。如果你想搞清楚究竟是否需要用到元类，那么你就不需要它。那些实际用到元类的人都非常清楚地知道他们需要做什么，而且根本不需要解释为什么要用元类。” —— Python 界的领袖 Tim Peters

元类的主要用途是创建API。一个典型的例子就是 Django ORM。

## 总结

Python 中的一些都是对象，要么是类的实例，要么是元类的实例，除了 type。type 是它自己的元类。

大多数情况我们可以通过 `monkey patching` 和 `class decorators` 来修改类，也是比较推荐的。

## 参考

* [What are metaclasses in Python?](https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python)
* [What is the difference between type and type.__new__ in python?](https://stackoverflow.com/questions/2608708/what-is-the-difference-between-type-and-type-new-in-python)
* [Django源码中的metaclass使用是如何兼容Python2和Python3的](https://www.the5fire.com/django-metaclass-compatibility.html)
* [Python 的 type 和 object 之间是怎么一种关系？](https://www.zhihu.com/question/38791962/answer/78172929)
