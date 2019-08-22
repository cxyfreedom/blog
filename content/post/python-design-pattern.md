---
title: Python 设计模式实现
date: 2019-04-21T20:58:10+08:00
description: ""
tags: ["Python", "设计模式"]
categories:
---
设计模式是指软件设计问题的推荐方案。设计模式一般是描述如何组织代码和使用最佳实践来解决常见的设计问题。

它是高层次的方案，并不关注具体的实现细节。

本文主要通过 Python 来简单介绍和实现不同的设计模式。

<!--more-->
## 创建型模式

### 工厂模式

工厂设计模式背后的思想是简化对象的创建，基于一个中心化函数来实现，易于追踪创建了哪些对象。通过将创建代码对象的代码和适用对象的代码解耦，工厂能够降低应用维护的复杂度。

工厂通常有两种形式：一种是工厂方法（Factory method），它是一个函数，对不同的输入参数返回不同的对象；第二种是抽象工厂，它是一组用于创建一系列相关事物对象的工厂方法。

Django 框架中使用工厂方法模式来创建表单字段。

#### 工厂方法

```python factory_method.py
import xml.etree.ElementTree as etree
import json


class JSONConnector:

    def __init__(self, filepath):
        self.data = dict()
        with open(filepath, mode='r', encoding='utf-8') as f:
            self.data = json.load(f)

    @property
    def parsed_data(self):
        return self.data


class XMLConnector:

    def __init__(self, filepath):
        self.tree = etree.parse(filepath)

    @property
    def parsed_data(self):
        return self.tree


def connection_factory(filepath):
    if filepath.endswith('json'):
        connector = JSONConnector
    elif filepath.endswith('xml'):
        connector = XMLConnector
    else:
        raise ValueError('Cannot connect to {}'.format(filepath))
    return connector(filepath)


def connect_to(filepath):
    factory = None
    try:
        factory = connection_factory(filepath)
    except ValueError as ve:
        print(ve)
    return factory


def main():
    sqlite_factory = connect_to('data/person.sq3')
    print()

    xml_factory = connect_to('data/person.xml')
    xml_data = xml_factory.parsed_data
    liars = xml_data.findall(".//{}[{}='{}']".format('person',
                                                     'lastName', 'Liar'))
    print(f'found: {len(liars)} persons')
    for liar in liars:
        print(f"first name: {liar.find('firstName').text}")
        print(f"last name: {liar.find('lastName').text}")
        [print(f"phone number ({p.attrib['type']})", p.text) for p in liar.find('phoneNumbers')]

    print()

    json_factory = connect_to('data/donut.json')
    json_data = json_factory.parsed_data
    print(f'found: {len(json_data)} donuts')
    for donut in json_data:
        print(f"name: {donut['name']}")
        print(f"price: ${donut['ppu']}")
        [print(f"topping: {t['id']} {t['type']}") for t in donut['topping']]


if __name__ == '__main__':
    main()
```

#### 抽象工厂

抽象工厂是工厂方法模式的一种泛化，它能让对象的创建更容易追踪。通过将对象的创建和使用解耦，还能够优化内存占用。

通常情况下一开始使用工厂方法，因为它更加简单。如果后来发现应用需要许多工厂方法，那么将创建一系列对象的过程合并在一起更合理，从而引入抽象工厂。

```python abstract_factory.py
class Frog:

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

    def interact_with(self, obstacle):
        print(f'{self} the Frog encounters {obstacle} and {obstacle.action()}!')


class Bug:

    def __str__(self):
        return 'a bug'

    def action(self):
        return 'eats it'


class FrogWorld:

    def __init__(self, name):
        print(self)
        self.player_name = name

    def __str__(self):
        return '\n\n\t------ Frog World ———'

    def make_character(self):
        return Frog(self.player_name)

    def make_obstacle(self):
        return Bug()


class Wizard:

    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name

    def interact_with(self, obstacle):
        print('{} the Wizard battles against {} and {}!'.format(self, obstacle,
                                                                obstacle.action()))


class Ork:

    def __str__(self):
        return 'an evil ork'

    def action(self):
        return 'kills it'


class WizardWorld:

    def __init__(self, name):
        print(self)
        self.player_name = name

    def __str__(self):
        return '\n\n\t------ Wizard World ———'

    def make_character(self):
        return Wizard(self.player_name)

    def make_obstacle(self):
        return Ork()


class GameEnvironment:

    def __init__(self, factory):
        self.hero = factory.make_character()
        self.obstacle = factory.make_obstacle()

    def play(self):
        self.hero.interact_with(self.obstacle)


def validate_age(name):
    try:
        age = input(f'Welcome {name}. How old are you? ')
        age = int(age)
    except ValueError as err:
        print(f"Age {age} is invalid, please try again…")
        return False, age
    return True, age


def main():
    name = input("Hello. What's your name? ")
    valid_input = False
    while not valid_input:
        valid_input, age = validate_age(name)
    game = FrogWorld if age < 18 else WizardWorld
    environment = GameEnvironment(game(name))
    environment.play()


if __name__ == '__main__':
    main()
```

### 建造者模式

建造者模式将一个复杂对象的构造过程与其表现分离，这样，同一个构造过程可用于创建多个不同的表现。该模式中，有两个参与者：建造者（builder）和指挥者（director）。建造者负责创建复杂对象的各个组成部分，指挥者使用一个建造者实例控制建造的过程。**在 HTML 例子中，这些组成部 分是页面标题、文本标题、内容主体及页脚。指挥者使用一个建造者实例控制建造的过程。对于 HTML 示例，这是指调用建造者的函数设置页面标题、文本标题等。使用不同的建造者实例让我 们可以创建不同的 HTML 页面，而无需变更指挥者的代码**。

在快餐店中使用的就是建造者设计模式，即使存在多种汉堡包和不同包装，准备一个汉堡包和打包的流程都是相同的。指挥者是出纳员，将需要准备的餐品指令传达给工作人员；建造者是工作人员中的个体，关注具体的顺序。

建造者模式和工厂模式的区别是，在工厂模式下，会立即返回一个创建好的对象；在建造者模式下，仅在需要时客户端代码才显式地请求指挥者返回最总的对象。

用新电脑的例子类比可能更加直观。假设需要购买一台新电脑，如果决定购买一台特定的预配置的电脑型号，则是在使用工厂模式。

```python apple-factory.py
MINI14 = '1.4GHz Mac mini'


class AppleFactory:
    class MacMini14:

        def __init__(self):
            self.memory = 4  # 单位为GB
            self.hdd = 500  # 单位为GB
            self.gpu = 'Intel HD Graphics 5000'

        def __str__(self):
            info = (f'Model: {MINI14}',
                    f'Memory: {self.memory}GB',
                    f'Hard Disk: {self.hdd}GB',
                    f'Graphics Card: {self.gpu}')
            return '\n'.join(info)

    def build_computer(self, model):
        if model == MINI14:
            return self.MacMini14()
        else:
            print(f"I dont't know how to build {model}")


if __name__ == '__main__':
    afac = AppleFactory()
    mac_mini = afac.build_computer(MINI14)
    print(mac_mini)
```

如果选择购买一台定制的 PC，使用的就是建造者模式。你是指挥者，向制造商（建造者）提供指令说明心中理想的电脑规格。

```python computer-builder.py
class Computer:

    def __init__(self, serial_number):
        self.serial = serial_number
        self.memory = None  # 单位为GB
        self.hdd = None  # 单位为GB
        self.gpu = None

    def __str__(self):
        info = (f'Memory: {self.memory}GB',
                f'Hard Disk: {self.hdd}GB',
                f'Graphics Card: {self.gpu}')
        return '\n'.join(info)


class ComputerBuilder:

    def __init__(self):
        self.computer = Computer('AG23385193')

    def configure_memory(self, amount):
        self.computer.memory = amount

    def configure_hdd(self, amount):
        self.computer.hdd = amount

    def configure_gpu(self, gpu_model):
        self.computer.gpu = gpu_model


class HardwareEngineer:

    def __init__(self):
        self.builder = None

    def construct_computer(self, memory, hdd, gpu):
        self.builder = ComputerBuilder()
        [step for step in (self.builder.configure_memory(memory),
                           self.builder.configure_hdd(hdd),
                           self.builder.configure_gpu(gpu))]

    @property
    def computer(self):
        return self.builder.computer


def main():
    engineer = HardwareEngineer()
    engineer.construct_computer(hdd=500, memory=8, gpu='GeForce GTX 650 Ti')
    computer = engineer.computer
    print(computer)


if __name__ == '__main__':
    main()
```

> 提示：通过嵌套类的方式，可以禁止直接实例化一个类。

#### 实现

通过建造者设计模式来实现一个披萨订购的应用。准备一个披萨需要多个步骤，且这些操作遵从特定的顺序。

```python builder.py
from enum import Enum
import time

PizzaProgress = Enum('PizzaProgress', 'queued preparation baking ready')
PizzaDough = Enum('PizzaDough', 'thin thick')
PizzaSauce = Enum('PizzaSauce', 'tomato creme_fraiche')
PizzaTopping = Enum('PizzaTopping',
                    'mozzarella double_mozzarella bacon ham mushrooms red_onion oregano')
STEP_DELAY = 3  # 单位为秒


class Pizza:

    def __init__(self, name):
        self.name = name
        self.dough = None
        self.sauce = None
        self.topping = []

    def __str__(self):
        return self.name

    def prepare_dough(self, dough):
        self.dough = dough
        print('preparing the {} dough of your {}...'.format(self.dough.name,
                                                            self))
        time.sleep(STEP_DELAY)
        print('done with the {} dough'.format(self.dough.name))


class MargaritaBuilder:

    def __init__(self):
        self.pizza = Pizza('margarita')
        self.progress = PizzaProgress.queued
        self.baking_time = 5  # 单位为秒

    def prepare_dough(self):
        self.progress = PizzaProgress.preparation
        self.pizza.prepare_dough(PizzaDough.thin)

    def add_sauce(self):
        print('adding the tomato sauce to your margarita...')
        self.pizza.sauce = PizzaSauce.tomato
        time.sleep(STEP_DELAY)
        print('done with the tomato sauce')

    def add_topping(self):
        print(
            'adding the topping (double mozzarella, oregano) to your margarita')
        self.pizza.topping.append([i for i in
                                   (PizzaTopping.double_mozzarella,
                                    PizzaTopping.oregano)])
        time.sleep(STEP_DELAY)
        print('done with the topping (double mozzarrella, oregano)')

    def bake(self):
        self.progress = PizzaProgress.baking
        print(f'baking your margarita for {self.baking_time} seconds')
        time.sleep(self.baking_time)
        self.progress = PizzaProgress.ready
        print('your margarita is ready')


class CreamyBaconBuilder:

    def __init__(self):
        self.pizza = Pizza('creamy bacon')
        self.progress = PizzaProgress.queued
        self.baking_time = 7  # 单位为秒

    def prepare_dough(self):
        self.progress = PizzaProgress.preparation
        self.pizza.prepare_dough(PizzaDough.thick)

    def add_sauce(self):
        print('adding the crème fraîche sauce to your creamy bacon')
        self.pizza.sauce = PizzaSauce.creme_fraiche
        time.sleep(STEP_DELAY)
        print('done with the crème fraîche sauce')

    def add_topping(self):
        print('adding the topping (mozzarella, bacon, ham, mushrooms, '
              'red onion, oregano) to your creamy bacon')
        self.pizza.topping.append([t for t in
                                   (
                                   PizzaTopping.mozzarella, PizzaTopping.bacon,
                                   PizzaTopping.ham, PizzaTopping.mushrooms,
                                   PizzaTopping.red_onion,
                                   PizzaTopping.oregano)])
        time.sleep(STEP_DELAY)
        print('done with the topping (mozzarella, bacon, ham, mushrooms, '
              'red onion, oregano)')

    def bake(self):
        self.progress = PizzaProgress.baking
        print(
            'baking your creamy bacon for {} seconds'.format(self.baking_time))
        time.sleep(self.baking_time)
        self.progress = PizzaProgress.ready
        print('your creamy bacon is ready')


class Waiter:

    def __init__(self):
        self.builder = None

    def construct_pizza(self, builder):
        self.builder = builder
        [step() for step in (builder.prepare_dough,
                             builder.add_sauce, builder.add_topping,
                             builder.bake)]

    @property
    def pizza(self):
        return self.builder.pizza


def validate_style(builders):
    try:
        pizza_style = input(
            'What pizza would you like, [m]argarita or [c]reamy bacon? ')
        builder = builders[pizza_style]()
        valid_input = True
    except KeyError as err:
        print('Sorry, only margarita (key m) and creamy bacon (key c) are available')
        return False, None
    return True, builder


def main():
    builders = dict(m=MargaritaBuilder, c=CreamyBaconBuilder)
    valid_input = False
    while not valid_input:
        valid_input, builder = validate_style(builders)
    print()
    waiter = Waiter()
    waiter.construct_pizza(builder)
    pizza = waiter.pizza
    print()
    print('Enjoy your {}!'.format(pizza))


if __name__ == '__main__':
    main()
```

还有一种建造者模式，它会链式地调用建造者方法，通过将建造者本身定义为内部类并从其每个设置器方法返回自身来实现。

```python build2.py
class Pizza:

    def __init__(self, builder):
        self.garlic = builder.garlic
        self.extra_cheese = builder.extra_cheese

    def __str__(self):
        garlic = 'yes' if self.garlic else 'no'
        cheese = 'yes' if self.extra_cheese else 'no'
        info = (f'Garlic: {garlic}', f'Extra cheese: {cheese}')
        return '\n'.join(info)

    class PizzaBuilder:

        def __init__(self):
            self.extra_cheese = False
            self.garlic = False

        def add_garlic(self):
            self.garlic = True
            return self

        def add_extra_cheese(self):
            self.extra_cheese = True
            return self

        def build(self):
            return Pizza(self)


if __name__ == '__main__':
    pizza = Pizza.PizzaBuilder().add_garlic().add_extra_cheese().build()
    print(pizza)
```

### 原型模式

原型设计模式（Prototype design pattern）帮助我们创建对象的克隆。在 Python 中，可以使用 `copy.deepcopy()` 函数来完成。

浅拷贝和深拷贝的区别在于：

* 浅拷贝构造一个新的复合对象后，会尽可能地将在原始对象中找到的对象的引用插入新对象中。
* 深拷贝构造一个新的复合对象后，会递归地将在原始对象中找到的对象的副本插入新对象中。

#### 实现

使用原型模式创建一个展示图书信息的应用。

```python prototype.py
import copy
from collections import OrderedDict


class Book:

    def __init__(self, name, authors, price, **rest):
        """rest的例子有：出版商，长度，标签，出版日期"""
        self.name = name
        self.authors = authors
        self.price = price  # 单位为美元
        self.__dict__.update(rest)

    def __str__(self):
        mylist = []
        ordered = OrderedDict(sorted(self.__dict__.items()))
        for i in ordered.keys():
            mylist.append(f'{i}: {ordered[i]}')
            if i == 'price':
                mylist.append('$')
            mylist.append('\n')
        return ''.join(mylist)


class Prototype:

    def __init__(self):
        self.objects = dict()

    def register(self, identifier, obj):
        self.objects[identifier] = obj

    def unregister(self, identifier):
        del self.objects[identifier]

    def clone(self, identifier, **attr):
        found = self.objects.get(identifier)
        if not found:
            raise ValueError(f'Incorrect object identifier: {identifier}')
        obj = copy.deepcopy(found)
        obj.__dict__.update(attr)
        return obj


def main():
    b1 = Book('The C Programming Language',
              ('Brian W. Kernighan', 'Dennis M.Ritchie'),
              price=118, publisher='Prentice Hall',
              length=228, publication_date='1978-02-22',
              tags=('C', 'programming', 'algorithms', 'data structures'))

    prototype = Prototype()
    cid = 'k&r-first'
    prototype.register(cid, b1)
    b2 = prototype.clone(cid, name='The C Programming Language(ANSI)',
                         price=48.99, length=274,
                         publication_date='1988-04-01', edition=2)

    for i in (b1, b2):
        print(i)
    print(f'ID b1 : {id(b1)} != ID b2 : {id(b2)}')


if __name__ == '__main__':
    main()
```

## 结构型模式

### 适配器模式

适配器模式（Adapter pattern）是一种结构型设计模式，帮助我们实现两个不兼容接口之间的兼容。这里的不兼容指的是，比如我们希望把一个旧组件用于新的系统中，但是不能修改代码或者访问代码等。此时，我们可以通过编写一个额外的代码层，该代码层包含让两个接口之间能够通信需要进行的所有修改。这个代码层就是适配器。

#### 实现

external.py 文件表示外部类：
```python
class Synthesizer:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f'the {self.name} synthesizer'

    def play(self):
        return 'is playing an electronic song'


class Human:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f'{self.name} the human'

    def speak(self):
        return 'says hello'
```

adapter.py 则是适配类：
```python
from external import Synthesizer, Human


class Computer:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f'the {self.name} computer'

    def execute(self):
        return 'executes a program'


class Adapter:
    def __init__(self, obj, adapted_methods):
        self.obj = obj
        self.__dict__.update(self.obj.__dict__)  # 将未适配内容委托给适配器类对象
        self.__dict__.update(adapted_methods)

    def __str__(self):
        return str(self.obj)


def main():
    objects = [Computer('Asus')]
    synth = Synthesizer('moog')
    objects.append(Adapter(synth, dict(execute=synth.play)))
    human = Human('Bob')
    objects.append(Adapter(human, dict(execute=human.speak)))

    for i in objects:
        print(f'Name: {i.name}')
        print(f'{str(i)} {i.execute()}')


if __name__ == '__main__':
    main()
```

### 装饰器模式

装饰器模式能够让我们动态地扩展一个对象的功能。

在现实生活中的实际例子，比如给枪加一个消音器，使用不同的照相机镜头等等。

在软件中，Django 框架就大量使用装饰器，比如视图装饰器，它可以用于以下几个用途：

* 限制某些 HTTP 请求对视图的访问
* 控制特定视图上的缓存行为
* 按单个视图控制压缩
* 基于特定 HTTP 请求头控制缓存

除了之外，装饰器还能应用在许多地方，尤其是实现横切关注点。数据校验、事务处理、缓存、日志、监控、调试、业务规则、压缩、加密等等。

#### 实现

```python
import functools


def memoize(fn):
    known = dict()

    @functools.wraps(fn)
    def memoizer(*args):
        if args not in known:
            known[args] = fn(*args)
        return known[args]

    return memoizer


@memoize
def nsum(n):
    """返回前 n 个数字的和"""
    assert (n >= 0), 'n must be >= 0'
    return n if n in (0, 1) else n + nsum(n - 1)


@memoize
def fibonacci(n):
    """返回斐波那契的第 n 个数"""
    assert (n >= 0), 'n must be >= 0'
    return n if n in (0, 1) else fibonacci(n - 1) + fibonacci(n - 2)


if __name__ == '__main__':
    from timeit import Timer

    measure = [
        {'exec': fibonacci(100), 'import': 'fibonacci', 'func': fibonacci},
        {'exec': nsum(200), 'import': 'nsum', 'func': nsum},
    ]

    for m in measure:
        t = Timer(f'{m["exec"]}', f'from __main__ import {m["import"]}')
        print(f'name: {m["func"].__name__}, doc: {m["func"].__doc__}, '
              f'executing: {m["exec"]}, time: {t.timeit()}')
```

如果希望在运行的时候不执行装饰器，可以通过 `func.__wrapped` 直接执行原始函数（前提是需要引入 wraps 模块）。

### 外观模式

外观模式有助于隐藏系统的内部复杂性，并通过一个简化的的接口向客户端暴露必要的部分。本质上，外观（Facade）是在已有复杂系统之上实现的一个抽象层。

使用外观模式通常是为一个复杂的系统提供单个简单的入口点。这样客户端代码通过简单地调用一个方法或者函数就能够使用一个系统。

#### 实现

模拟多服务进程实现操作系统。多服务进程的操作系统有一个极小的内核，称为微内核，它在特权模式下运行。系统的所有其他服务都遵从一种服务架构（驱动程序服务器、进程服务器、文件服务器等）。每个服务进程属于一个不同的内存地址空间，以用户模式在微内核之上运行。比如所有驱动程序都以用户模式在一个驱动服务进程之上运行。

```python facade.py
from enum import Enum
from abc import ABCMeta, abstractmethod

State = Enum('State', 'new running sleeping restart zombie')


class User:
    pass


class Process:
    pass


class File:
    pass


class Server(metaclass=ABCMeta):
    @abstractmethod
    def __init__(self):
        pass

    def __str__(self):
        return self.name

    @abstractmethod
    def kill(self, restart=True):
        pass


class FileServer(Server):
    def __init__(self):
        """初始化文件服务进程要求的操作"""
        self.name = 'FileServer'
        self.state = State.new

    def boot(self):
        """启动文件服务进程要求的操作"""
        print(f'booting the {self}')
        self.state = State.running

    def kill(self, restart=True):
        """终止文件服务进程要求的操作"""
        print(f'Killing the {self}')
        self.state = State.restart if restart else State.zombie

    def create_file(self, user, name, permissions):
        """检查访问权限的有效性、用户权限等"""
        print(f"trying to create the file '{name}' for user '{user}' "
              f"with permissions {permissions}")


class ProcessServer(Server):
    def __init__(self):
        """初始化进程服务进程要求的操作"""
        self.name = 'ProcessServer'
        self.state = State.new

    def boot(self):
        """启动进程服务进程要求的操作"""
        print(f'booting the {self}')
        self.state = State.running

    def kill(self, restart=True):
        """终止进程服务进程要求的操作"""
        print(f'Killing the {self}')
        self.state = State.restart if restart else State.zombie

    def create_process(self, user, name):
        """检查用户权限和生成PID等"""
        print(f"trying to create the process '{name}' for user '{user}'")


class WindowServer(Server):
    pass


class NetworkServer(Server):
    pass


class OperatingSystem:
    """外观"""

    def __init__(self):
        self.fs = FileServer()
        self.ps = ProcessServer()

    def start(self):
        [i.boot() for i in (self.fs, self.ps)]

    def create_file(self, user, name, permissions):
        return self.fs.create_file(user, name, permissions)

    def create_process(self, user, name):
        return self.ps.create_process(user, name)


def main():
    os = OperatingSystem()
    os.start()
    os.create_file('foo', 'hello', '-rw-r-r')
    os.create_process('bar', 'ls /tmp')


if __name__ == '__main__':
    main()
```

### 享元模式

享元模式通过为相似对象引入数据共享来最小化内存使用，提升性能。一个享元（Flyweight）就是一个包含状态独立的不可变数据的共享对象。

享元主要在优化性能和内存中使用，比如嵌入式系统和游戏等。

享元模式和 memoization 的区别在于，memoization 是一种优化技术，使用一个缓存来避免重复计算那些在更早的执行步骤中已经计算好的结果。并且可以应用于多种编程方式中。享元模式则是一种特定与面向对象编程优化的设计模式，关注的是共享对象数据。

#### 实现

模拟一小片果树的树林，小到能确保在单个终端页面中阅读整个输出。无论构造的森林有多大，内存分配都保持相同。

```python flyweight.py
import random
from enum import Enum

TreeType = Enum('TreeType', 'apple_tree cherry_tree peach_tree')


class Tree:
    pool = dict()

    def __new__(cls, tree_type):
        """将Tree类变为一个元类，元类支持自引用"""
        obj = cls.pool.get(tree_type, None)
        if not obj:
            obj = object.__new__(cls)
            cls.pool[tree_type] = obj
            obj.tree_type = tree_type
        return obj

    def render(self, age, x, y):
        print(
            f'render a tree of type {self.tree_type} and age {age} at ({x}, {y})')


def main():
    rnd = random.Random()
    age_min, age_max = 1, 30
    min_point, max_point = 0, 100
    tree_counter = 0

    for _ in range(10):
        t1 = Tree(TreeType.apple_tree)
        t1.render(rnd.randint(age_min, age_max),
                  rnd.randint(min_point, max_point),
                  rnd.randint(min_point, max_point))
        tree_counter += 1

    for _ in range(3):
        t2 = Tree(TreeType.cherry_tree)
        t2.render(rnd.randint(age_min, age_max),
                  rnd.randint(min_point, max_point),
                  rnd.randint(min_point, max_point))
        tree_counter += 1

    for _ in range(5):
        t3 = Tree(TreeType.peach_tree)
        t3.render(rnd.randint(age_min, age_max),
                  rnd.randint(min_point, max_point),
                  rnd.randint(min_point, max_point))
        tree_counter += 1

    print(f'trees rendered: {tree_counter}')
    print(f'trees actually created: {len(Tree.pool)}')

    t4 = Tree(TreeType.cherry_tree)
    t5 = Tree(TreeType.cherry_tree)
    t6 = Tree(TreeType.apple_tree)
    print(f'{id(t4)} == {id(t5)}? {id(t4) == id(t5)}')
    print(f'{id(t5)} == {id(t6)}? {id(t5) == id(t6)}')


if __name__ == '__main__':
    main()
```

### 模型-视图-控制器模式

关注点分离（Separation of Concerns，Soc）原则是将一个应用切分成不同的部分，每个部分解决一个单独的关注点。模型-视图-控制器（Model-View-Controller，MVC）模式是应用到面向对象编程的 Soc 原则。MVC 更多被认为是一种架构模式，而不是一种设计模式。

SoC原则在现实生活中的应用也很多。比如在一个餐馆中，服务员接收点菜单并为顾客上菜，但是饭菜由厨师烹饪。

许多流行的 Web 框架（Django、Rails 和 Yii）和应用框架（iPhone SDK、Android 和 QT）都使用了 MVC 或者其变种，其变种包括模型-模板-视图（Model-Template-View，MTV）、模式-视图-适配器（Model-View-Adapter，MVA）、模型-视图-演示者（Model-View-Presenter，MVP）。

可以将具有以下功能的模型视为智能模型。

* 包含所有的校验/业务规则/逻辑
* 处理应用的状态
* 访问应用数据（数据库、云或其他）
* 不依赖 UI

可以将符合以下条件的控制器视为瘦控制器。

* 在用户与视图交互时，更新模型
* 在模型改变时，更新视图
* 如果需要，在数据传递给模型/视图之前进行处理
* 不展示数据
* 不直接访问应用数据
* 不包含校验/业务规则/逻辑

可以将符合以下条件的视图视为傻瓜视图。

* 展示数据
* 允许用户与其交互
* 仅做最小的数据处理，通常由一种模板语言提供处理能力(例如，使用简单的变量和循环控制）
* 不存储任何数据
* 不直接访问应用数据
* 不包含校验/业务规则/逻辑

#### 实现

```python mvc.py
# 数据通常是存储在数据库、文件等地方，只有模型能够直接访问。
quotes = ('A man is not complete until he is married. Then he is finished.',
          'As I said before, I never repeat myself.',
          'Behind a successful man is an exhausted woman.',
          'Black holes really suck...', 'Facts are stubborn things.')


class QuoteModel:
    def get_quote(self, n):
        try:
            value = quotes[n]
        except IndexError as err:
            value = 'Not found!'
        return value


class QuoteTerminalView:
    def show(self, quote):
        """在屏幕上输出一句名人名言"""
        print(f'And the quote is "{quote}"')

    def error(self, msg):
        """输出一条错误信息"""
        print(f'Error: {msg}')

    def select_quote(self):
        """读取用户的选择"""
        return input('Which quote number would you like to see? ')


class QuoteTerminalController:
    def __init__(self):
        self.model = QuoteModel()
        self.view = QuoteTerminalView()

    def run(self):
        valid_input = False
        while not valid_input:
            n = self.view.select_quote()
            try:
                n = int(n)
            except ValueError as err:
                self.view.error(f"Incorrect index '{n}'")
            else:
                valid_input = True
        quote = self.model.get_quote(n)
        self.view.show(quote)


def main():
    controller = QuoteTerminalController()
    while True:
        controller.run()


if __name__ == '__main__':
    main()
```

### 代理模式

代理模式，通过代理对象在访问实际对象之前执行重要的操作的一种设计模式。比较知名的代理类型有：

* 远程代理：实际存在于不同地址空间的对象在本地的代理者。
* 虚拟代理：用于懒初始化，将一个计算量对象的创建延迟到真正需要的时候进行。例如 Django 中的 LazyLoad。
* 保护/防护代理：控制对敏感对象的访问。
* 智能（引用）代理：在对象被访问时，执行额外的动作。比如引用计数和线程安全检查。

#### 懒初始化

```python lazy.py
class LazyProperty:
    def __init__(self, method):
        self.method = method
        self.method_name = self.method.__name__
        # print(f'function overridden: {self.method}')
        # print(f"function's name: {self.method_name}")

    def __get__(self, instance, owner):
        if not instance:
            return
        value = self.method(instance)
        # print(f'value {value}')
        # 给 instance 增加一个 self.method_name 的属性，这个属性和对应的值会写
        # 入到 instance.__dict__ 中，下次获取时首先会从 instance__dict__ 中读取
        setattr(instance, self.method_name, value)
        return value


class Test:
    def __init__(self):
        self.x = 'foo'
        self.y = 'bar'
        self._resource = None

    @LazyProperty
    def resource(self):
        print(f'initializing self._resource which is {self._resource}')
        self._resource = tuple(range(5))
        return self._resource


def main():
    t = Test()
    print(t.x)
    print(t.y)
    print(t.resource)
    print(t.resource)


if __name__ == '__main__':
    main()
```

在 OOP 中有两种基本的、不同类型的懒初始化：

* 在实例级：这意味着会对一个对象的特性进行懒初始化，但该特性有一个对象作用域。同一个类的每个实例都有自己的特性副本。
* 在类级或模块级：所有实例共享同一个特性，特性是懒初始化的。

代理模式的应用案例也有许多：

* 在使用私有网络或者云搭建一个分布式系统时，在分布式系统中，一些对象存在于本地内存中，一些对象存在于远程计算机的内存中。如果我们不想本地代码关心两者的区别，可以创建一个远程代理来隐藏/封装。
* 使用虚拟代理引入懒初始化，在真正需要对象的时候才创建，能够明显提高性能。
* 用于检查一个用户是否有足够权限访问某个信息片段。
* ORM API 也是一个代理的例子，ORM 是关系型数据库的代理，数据库可以部署在任何地方。

#### 实现

实现一个简单的保护代理来查看和添加用户。

```python proxy.py
class SensitiveInfo(metaclass=ABCMeta):
    @abstractmethod
    def __init__(self):
        self.users = ['nick', 'tom', 'ben', 'mike']

    @abstractmethod
    def read(self):
        print(f'There are {len(self.users)} users: {" ".join(self.users)}')

    @abstractmethod
    def add(self, user):
        self.users.append(user)
        print(f'Added user: {user}')


class Info(SensitiveInfo):
    def __init__(self):
        super().__init__()
        self.secret = 'secret'

    def read(self):
        super().read()

    def add(self, user):
        sec = input('what is the secret? ')
        super().add(user) if sec == self.secret else print(
            "That's wrong!")


def main():
    info = Info()

    while True:
        print('1. read list |==| 2. add user |==| 3. quit')
        key = input('choose option: ')
        if key == '1':
            info.read()
        elif key == '2':
            name = input('choose username: ')
            info.add(name)
        elif key == '3':
            exit()
        else:
            print(f'unknown option: {key}')


if __name__ == '__main__':
    main()
```

## 行为型模式

### 责任链模式

责任链模式用于让多个对象来处理单个请求时，或者用于预先不知道应该由哪个对象（来自某个对象链）来处理某个特定请求时，其原则如下：

1. 存在一个对象链（链表、树或其他数据结构）。
2. 首先将请求发送给链中的第一个对象。
3. 对象决定是否要处理该请求。
4. 对象将请求转发给下一个对象。
5. 重复该过程，知道达到链尾。

#### 实现

实现一个简单的事件系统。

```python chain.py
class Event:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name


class Widget:
    def __init__(self, parent=None):
        self.parent = parent

    def handle(self, event):
        handler = f'handle_{event}'
        if hasattr(self, handler):
            method = getattr(self, handler)
            method(event)
        elif self.parent:
            self.parent.handle(event)
        elif hasattr(self, 'handle_default'):
            self.handle_default(event)


class MainWindow(Widget):
    def handle_close(self, event):
        print(f'MainWindow: {event}')

    def handle_default(self, event):
        print(f'MainWindow Default: {event}')


class SendDialog(Widget):
    def handle_paint(self, event):
        print(f'SendDialog: {event}')


class MsgText(Widget):
    def handle_down(self, event):
        print(f'MsgText: {event}')


def main():
    mw = MainWindow()
    sd = SendDialog(mw)
    msg = MsgText(sd)

    for e in ('down', 'paint', 'unhandled', 'close'):
        evt = Event(e)
        print(f'\nSending event -{evt}- to MainWindow')
        mw.handle(evt)
        print(f'Sending event -{evt}- to SendDialog')
        sd.handle(evt)
        print(f'Sending event -{evt}- to MsgText')
        msg.handle(evt)


if __name__ == '__main__':
    main()
```

适合责任链模式的系统有基于事件的系统、采购系统和运输系统。

### 命令模式

命令设计模式帮助我们将一个操作（撤销、重做、复制、粘贴等）封装成一个对象。这意味着创建一个类，包含实现该操作所需要的所有逻辑和方法。这样做的好处在于：

* 不需要直接执行一个命令，命令可以按照希望去执行。
* 解耦，不需要知道命令的实现细节。
* 可以将多个命令组织起来，顺序调用。

在实际软件中，PyQt 包含一个 QAction 类，它将一个动作建模为一个命令，对每个动作都支持额外的可选信息。

#### 实现

使用命令模式实现最基本的文件操作工具。包括：

* 创建一个文件，并随意写入一个字符串
* 读取一个文件的内容
* 重命名一个文件
* 删除一个文件

```python command.py
import os

verbose = True


class RenameFile:
    def __init__(self, path_src, path_dest):
        self.src, self.dest = path_src, path_dest

    def execute(self):
        if verbose:
            print(f"[renaming '{self.src}' to '{self.dest}']")
        os.rename(self.src, self.dest)

    def undo(self):
        if verbose:
            print(f"[renaming '{self.dest}' back to '{self.src}']")
        os.rename(self.dest, self.src)


class CreateFile:
    def __init__(self, path, txt='hello world\n'):
        self.path, self.txt = path, txt

    def execute(self):
        if verbose:
            print(f"[creating file '{self.path}']")
        with open(self.path, mode='w', encoding='utf-8') as out_file:
            out_file.write(self.txt)

    def undo(self):
        delete_file(self.path)


class ReadFile:
    def __init__(self, path):
        self.path = path

    def execute(self):
        if verbose:
            print("[reading file '{self.path}']")
        with open(self.path, mode='r', encoding='utf-8') as in_file:
            print(in_file.read(), end='')


def delete_file(path):
    if verbose:
        print(f"deleting file '{path}'")
    os.remove(path)


def main():
    orig_name, new_name = 'file1', 'file2'

    commands = []
    for ⌘ in CreateFile(orig_name), ReadFile(orig_name), RenameFile(
            orig_name, new_name):
        commands.append(⌘)

    [c.execute() for c in commands]

    answer = input('reverse the executed commands? [y//n]')

    if answer not in 'Yy':
        print(f'the result is {new_name}')
        exit()

    for c in reversed(commands):
        try:
            c.undo()
        except AttributeError as e:
            pass


if __name__ == '__main__':
    main()
```

### 解释器模式

解释器模式用于为高级用户和领域专家提供一个类编程的框架，但没有暴露出编程语言那样的复杂性。主要通过实现一个 DSL 来达到目的。

DSL（Domain Specific Language）是一种针对一个特定领域的有限表达能力的计算机语言。DSL 分为内部 DSL 和外部 DSL。

内部 DSL 构建在一种诉诸编程语言之上。比如使用 Python 解决线性方程组的一种语言。内部 DSL 的优势在于不必担心创建、编译及语法解析。劣势是会受限于宿主语言的特性。

外部 DSL 不依赖某种宿主语言，但是创建过程非常复杂。

解释器模式仅与内部 DSL 相关。

#### 实现

创建一种内部 DSL 控制一个智能屋。这里我们需要使用 `Pyparsing` 库。

```python interpreter.py
from pyparsing import Word, OneOrMore, Optional, Group, Suppress, alphanums


class Gate:
    def __init__(self):
        self.is_open = False

    def __str__(self):
        return 'open' if self.is_open else 'closed'

    def open(self):
        print('opening the gate')
        self.is_open = True

    def close(self):
        print('closing the gate')
        self.is_open = False


class Garage:
    def __init__(self):
        self.is_open = False

    def __str__(self):
        return 'open' if self.is_open else 'closed'

    def open(self):
        print('opening the garage')
        self.is_open = True

    def close(self):
        print('closing the garage')
        self.is_open = False


class Aircondition:
    def __init__(self):
        self.is_on = False

    def __str__(self):
        return 'on' if self.is_on else 'off'

    def turn_on(self):
        print('turning on the aircondition')
        self.is_on = True

    def turn_off(self):
        print('turning off the aircondition')
        self.is_on = False


class Heating:
    def __init__(self):
        self.is_on = False

    def __str__(self):
        return 'on' if self.is_on else 'off'

    def turn_on(self):
        print('turning on the heating')
        self.is_on = True

    def turn_off(self):
        print('turning off the heating')
        self.is_on = False


class Boiler:
    def __init__(self):
        self.temperature = 83

    def __str__(self):
        return f'boiler temperature {self.temperature}'

    def increase_temperature(self, amount):
        print(f"increasing the boiler's temperature by {amount} degrees")
        self.temperature += amount

    def decrease_temperature(self, amount):
        print(f"decreasing the boiler's temperature by {amount} degrees")
        self.temperature -= amount


class Fridge:
    def __init__(self):
        self.temperature = 2

    def __str__(self):
        return f'fridge temperature {self.temperature}'

    def increase_temperature(self, amount):
        print(f"increasing the fridge's temperature by {amount} degrees")
        self.temperature += amount

    def decrease_temperature(self, amount):
        print(f"decreasing the boiler's temperature by {amount} degrees")
        self.temperature -= amount


def main():
    word = Word(alphanums)
    command = Group(OneOrMore(word))
    token = Suppress("->")
    device = Group(OneOrMore(word))
    argument = Group(OneOrMore(word))
    event = command + token + device + Optional(token + argument)

    gate = Gate()
    garage = Garage()
    airco = Aircondition()
    heating = Heating()
    boiler = Boiler()
    fridge = Fridge()

    tests = ('open -> gate',
             'close -> garage',
             'turn on -> aircondition',
             'turn off -> heating',
             'increase -> boiler temperature -> 5 degrees',
             'decrease -> fridge temperature -> 2 degrees')
    open_actions = {'gate': gate.open,
                    'garage': garage.open,
                    'aircondition': airco.turn_on,
                    'heating': heating.turn_on,
                    'boiler temperature': boiler.increase_temperature,
                    'fridge temperature': fridge.increase_temperature}
    close_actions = {'gate': gate.close,
                     'garage': garage.close,
                     'aircondition': airco.turn_off,
                     'heating': heating.turn_off,
                     'boiler temperature': boiler.decrease_temperature,
                     'fridge temperature': fridge.decrease_temperature}

    for t in tests:
        if len(event.parseString(t)) == 2:  # 没有参数
            ⌘, dev = event.parseString(t)
            ⌘_str, dev_str = ' '.join(⌘), ' '.join(dev)
            if 'open' in ⌘_str or 'turn on' in ⌘_str:
                open_actions[dev_str]()
            elif 'close' in ⌘_str or 'turn off' in ⌘_str:
                close_actions[dev_str]()
        elif len(event.parseString(t)) == 3:  # 有参数
            ⌘, dev, arg = event.parseString(t)
            ⌘_str, dev_str, arg_str = ' '.join(⌘), ' '.join(dev), ' '.join(
                arg)
            num_arg = 0
            try:
                num_arg = int(arg_str.split()[0])  # 抽取数值部分
            except ValueError as err:
                print("expected number but got: '{}'".format(arg_str[0]))
            if 'increase' in ⌘_str and num_arg > 0:
                open_actions[dev_str](num_arg)
            elif 'decrease' in ⌘_str and num_arg > 0:
                close_actions[dev_str](num_arg)


if __name__ == '__main__':
    main()
```

### 观察者模式

观察者模式描述单个对象（发布者，又称为主持者或者可观察者）与一个或多个对象（订阅者，又称为观察者）之间的发布-订阅关系。在 MVC 中，发布者是模型，订阅者是视图，RSS 也是一种观察者模式。

#### 实现

模拟一个数据格式化程序。默认以十进制格式展示一个数值，可以添加/注册更多的格式化程序，每次更新格式化程序的值时，已注册的格式化程序就会收到通知并展示新的格式化后的值。

```python observer.py
class Publisher:
    def __init__(self):
        self.observers = []

    def add(self, observer):
        if observer not in self.observers:
            self.observers.append(observer)
        else:
            print(f'Failed to add: {observer}')

    def remove(self, observer):
        try:
            self.observers.remove(observer)
        except ValueError:
            print(f'Failed to remove: {observer}')

    def notify(self):
        [o.notify(self) for o in self.observers]


class DefaultFormatter(Publisher):
    def __init__(self, name):
        super().__init__()
        self.name = name
        self._data = 0

    def __str__(self):
        return f"{type(self).__name__}: '{self.name}' has data = {self._data}"

    @property
    def data(self):
        return self._data

    @data.setter
    def data(self, new_value):
        try:
            self._data = int(new_value)
        except ValueError as e:
            print(f'Error: {e}')
        else:
            self.notify()


class HexFormatter:
    def notify(self, publisher):
        print(f"{type(self).__name__}: '{publisher.name}' has now hex data "
              f"= {hex(publisher.data)}")


class BinaryFormatter:
    def notify(self, publisher):
        print(f"{type(self).__name__}: '{publisher.name}' has now bin data "
              f"= {bin(publisher.data)}")


def main():
    df = DefaultFormatter('test1')
    print(df)

    print()
    hf = HexFormatter()
    df.add(hf)
    df.data = 3
    print(df)

    print()
    bf = BinaryFormatter()
    df.add(bf)
    df.data = 21
    print(df)

    print()
    df.remove(hf)
    df.data = 40
    print(df)

    print()
    df.remove(hf)
    df.add(bf)

    df.data = 'hello'
    print(df)

    print()
    df.data = 15.8
    print(df)


if __name__ == '__main__':
    main()
```

### 状态模式

状态机是一个抽象机器，有两个关键部分，状态和转换。状态是指系统的当前（激活）状态。比如收音机可能有两个状态 FM 或 AM，另一个可能的状态是从 FM/AM 切换到另一个。转换是指从一个状态切换到另一个状态，因某个事件或条件的触发而开始。通常，在一次转换发生之前或之后会执行一个或一组动作。

状态机可以通过图来表现，其中每个状态都是一个节点，每个转换都是两个节点之间的边。

状态机用于解决许多计算机问题的非计算机问题，其中包括交通灯、停车计时器、硬件设计和编程语言解析等。

#### 实现

创建一个状态机，模拟一个进程的不同状态以及它们之间的转换。

状态设计模式通常使用一个父 State 类和许多派生的 ConcreteState 类来实现，父类包含所有状态共同的功能，每个派生类则仅包含特定状态要求的功能。

这里利用 `state_machine` 这个第三方模块，通过 `pip install state_machine`安装。

```python state.py
from state_machine import (State, Event, acts_as_state_machine, after, before,
                           InvalidStateTransition)


@acts_as_state_machine
class Process:
    created = State(initial=True)
    waiting = State()
    running = State()
    terminated = State()
    blocked = State()
    swapped_out_waiting = State()
    swapped_out_blocked = State()

    wait = Event(from_states=(created, running, blocked, swapped_out_waiting),
                 to_state=waiting)
    run = Event(from_states=waiting, to_state=running)
    terminate = Event(from_states=running, to_state=terminated)
    block = Event(from_states=(running, swapped_out_blocked), to_state=blocked)
    swap_wait = Event(from_states=waiting, to_state=swapped_out_waiting)
    swap_block = Event(from_states=blocked, to_state=swapped_out_blocked)

    def __init__(self, name):
        self.name = name

    @after('wait')
    def wait_info(self):
        print(f'{self.name} entered waiting mode')

    @after('run')
    def run_info(self):
        print(f'{self.name} is running')

    @before('terminate')
    def terminate_info(self):
        print(f'{self.name} terminate')

    @after('block')
    def block_info(self):
        print(f'{self.name} is blocked')

    @after('swap_wait')
    def swap_wait_info(self):
        print(f'{self.name} is swapped out and waiting')

    @after('swap_block')
    def swap_block_info(self):
        print(f'{self.name} is swapped out and blocked')


def transition(process, event, event_name):
    try:
        event()
    except InvalidStateTransition as err:
        print(f'Error: transition of {process.name} '
              f'from {process.current_state} to {event_name} failed')


def state_info(process):
    print(f'state of {process.name}: {process.current_state}')


def main():
    RUNNING = 'running'
    WAITING = 'waiting'
    BLOCKED = 'blocked'
    TERMINATED = 'terminated'

    p1, p2 = Process('process1'), Process('process2')
    [state_info(p) for p in (p1, p2)]

    print()
    transition(p1, p1.wait, WAITING)
    transition(p2, p2.terminate, TERMINATED)
    [state_info(p) for p in (p1, p2)]

    print()
    transition(p1, p1.run, RUNNING)
    transition(p2, p2.wait, WAITING)
    [state_info(p) for p in (p1, p2)]

    print()
    transition(p2, p2.run, RUNNING)
    [state_info(p) for p in (p1, p2)]

    print()
    [transition(p, p.block, BLOCKED) for p in (p1, p2)]
    [state_info(p) for p in (p1, p2)]
    
    print()
    [transition(p, p.terminate, TERMINATED) for p in (p1, p2)]
    [state_info(p) for p in (p1, p2)]


if __name__ == '__main__':
    main()
```

### 策略模式

策略模式（Strategy pattern）鼓励使用多种算法来解决一个问题，其特性能够在运行时透明地切换算法。

在 Python 中的 `sorted` 函数就是使用策略模式的例子。

#### 实现

实现一个算法来检测在一个字符串中是否所有字符是唯一的。这里我们通过对字符串进行排序并逐对比较所有字符。

```python strategy.py
import time

SLOW = 3  # 单位为秒
LIMIT = 5  # 字符数
WARNING = 'too bad, you picked the slow algorithm :('


def pairs(seq):
    n = len(seq)
    for i in range(n):
        yield seq[i], seq[(i + 1) % n]


def allUniqueSort(s):
    if len(s) == 1:
        return True
    if len(s) > LIMIT:
        print(WARNING)
        time.sleep(SLOW)
    strStr = sorted(s)
    for (c1, c2) in pairs(strStr):
        if c1 == c2:
            return False
    return True


def allUniqueSet(s):
    if len(s) < LIMIT:
        print(WARNING)
        time.sleep(SLOW)

    return True if len(set(s)) == len(s) else False


def allUnique(s, strategy):
    return strategy(s)


def main():
    while True:
        word = None
        while not word:
            word = input('Insert word (type quit to exit)> ')
            if word == 'quit':
                print('bye')
                return

            strategy_picked = None
            strategies = {'1': allUniqueSet, '2': allUniqueSort}
            while strategy_picked not in strategies.keys():
                strategy_picked = input(
                    'Choose strategy: [1] Use a set, [2] Sort and pair> ')

                try:
                    strategy = strategies[strategy_picked]
                    print(f'allUnique({word}): {allUnique(word, strategy)}')
                except KeyError as err:
                    print(f'Incorrect option: {strategy_picked}')


if __name__ == '__main__':
    main()
```

通常情况下，我们使用的策略不应该由用户来选择。策略模式的要点是可以透明地使用不同的算法。

### 模板模式

在实现结构相近的算法时，可以使用模板模式来消除冗余代码。具体实现方式是使用动作/钩子函数来完成代码重复的消除。

比如 BFS 和 DFS 两个算法很相近，我们可以通过原型模式来消除代码冗余。

```python graph_template.py
def traverse(graph, start, end, action):
    path = []
    visited = [start]
    while visited:
        current = visited.pop(0)
        if current not in path:
            path.append(current)
            if current == end:
                return True, path
            # 两个顶点不相连，则跳过
            if current not in graph:
                continue
        visited = action(visited, graph[current])
    return False, path


def extend_bfs_path(visited, current):
    return visited + current


def extend_dfs_path(visited, current):
    return current + visited


def main():
    graph = {
        'Frankfurt': ['Mannheim', 'Wurzburg', 'Kassel'],
        'Mannheim': ['Karlsruhe'],
        'Karlsruhe': ['Augsburg'],
        'Augsburg': ['Munchen'],
        'Wurzburg': ['Erfurt', 'Nurnberg'],
        'Nurnberg': ['Stuttgart', 'Munchen'],
        'Kassel': ['Munchen'],
        'Erfurt': [],
        'Stuttgart': [],
        'Munchen': []
    }

    bfs_path = traverse(graph, 'Frankfurt', 'Nurnberg', extend_bfs_path)
    dfs_path = traverse(graph, 'Frankfurt', 'Nurnberg', extend_dfs_path)
    print('bfs Frankfurt-Nurnberg: {}'.format(
        bfs_path[1] if bfs_path[0] else 'Not found'))
    print('dfs Frankfurt-Nurnberg: {}'.format(
        dfs_path[1] if dfs_path[0] else 'Not found'))

    bfs_nopath = traverse(graph, 'Wurzburg', 'Kassel', extend_bfs_path)
    print('bfs Wurzburg-Kassel: {}'.format(
        bfs_nopath[1] if bfs_nopath[0] else 'Not found'))
    dfs_nopath = traverse(graph, 'Wurzburg', 'Kassel', extend_dfs_path)
    print('dfs Wurzburg-Kassel: {}'.format(
        dfs_nopath[1] if dfs_nopath[0] else 'Not found'))


if __name__ == '__main__':
    main()
```

#### 实现

实现一个横幅生成器。需要安装第三方库 `cowpy`。

```python template.py
from cowpy import cow


def dots_style(msg):
    msg = msg.capitalize()
    msg = '.' * 10 + msg + '.' * 10
    return msg


def admire_style(msg):
    msg = msg.upper()
    return '!'.join(msg)


def cow_style(msg):
    msg = cow.milk_random_cow(msg)
    return msg


def generate_banner(msg, style=dots_style):
    print('-- start of banner --')
    print(style(msg))
    print('-- end of banner --\n\n')


def main():
    msg = 'happy coding'
    [generate_banner(msg, style) for style in
     (dots_style, admire_style, cow_style)]


if __name__ == '__main__':
    main()
```

## 参考

* 《精通 Python 设计模式》