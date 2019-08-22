---
title: Python源码解析——可重入锁RLock
date: 2018-08-20 20:07:47
tags: ["Python", "源码"]
categories: ["Python"]
---
本文主要针对 Python 中的可重入锁 RLock 的源码剖析。

<!--more-->
## 源码（带注释）

可重入锁 RLock 的源码在 `threading.py` 文件中

```python
def RLock(*args, **kwargs):
    """
    该工厂函数返回一个新的可重入锁。
    一个可重入锁必须由创建它的线程释放。一旦一个线程获得了一个可重入锁，该线程可用无阻塞的再次获取。每次获取锁后必须进行释放。
    Python支持用C语言实现的 RLock 锁和用Python本身实现的 RLock 锁，默认使用的是C语言版本的
    """
    if _CRLock is None:
        return _PyRLock(*args, **kwargs)
    return _CRLock(*args, **kwargs)

class _RLock:
    # RLock的核心即 acquire 和 release 这两个方法

    def __init__(self):
        self._block = _allocate_lock()  # 分配一个锁
        self._owner = None  # RLock对象所属的线程pid
        self._count = 0     # 锁计数器，对于RLock对象所在线程，每获取一次就加一，相对的每次释放就减一，当减到零时，就会释放内部创建的锁，这样其他线程就可以继续获得这个锁。

    def acquire(self, blocking=True, timeout=-1):
        me = get_ident()    # 获取当前线程的pid
        if self._owner == me:   # 如果当前线程的pid是RLock对象所在的线程，那么对计数器进行加一操作
            self._count += 1
            return 1
        # 如果不满足上述条件：
        # 1. 当前线程非RLock对象所在线程
        # 2. RLock对象还未持有锁，即 self.owner = None
        # 那么当 blocking=True 时，当前线程被阻塞，直到持有锁的线程将锁释放后，rc = True
        # 当 blocking=False 时，可以非阻塞的获取。如果获取锁成功，rc = True；获取失败，rc = False
        rc = self._block.acquire(blocking, timeout)
        if rc:
            self._owner = me    # 记录持有锁的线程的pid
            self._count = 1     # 将计数器重置到1
        return rc

    def release(self):
        if self._owner != get_ident():  # 如果持有锁的线程非当前线程，则抛异常
            raise RuntimeError("cannot release un-acquired lock")
        self._count = count = self._count - 1   # 每次释放对计数器进行减一
        if not count:   # 如果计数器减到0，那么释放RLock内部的锁，此时其他线程就可以获取到锁
            self._owner = None    # 还原RLock对象持有锁的拥有者None
            self._block.release() # 释放锁
```

## acquire 方法流程图

```flow
st=>start: 调用 acquire
e1=>end: return 1
e2=>end: return rc
op1=>operation: 获取当前线程的pid
cond1=>condition: 当前线程是否为RLock对象所属线程？
op2=>operation: 计数器_count加1
op3=>operation: 获取内部互斥锁
cond2=>condition: 获取锁是否成功？
op4=>operation: 将当前线程的pid赋给_owner，并将_count设置为1

st->op1->cond1
cond1(yes)->op2->e1
cond1(no, left)->op3->cond2
cond2(yes)->op4->e2
cond2(no, left)->e2
```

## release 方法流程图

```flow
st=>start: 调用 release
e1=>end: 抛出异常
e2=>end: return
op1=>operation: 获取当前线程的pid
cond1=>condition: 当前线程是否为RLock对象所属线程？
op2=>operation: 计数器_count减1
cond2=>condition: count是否为0？
op3=>operation: _owner还原为None，并释放内部锁

st->op1->cond1
cond1(yes, left)->op2->cond2
cond1(no)->e1
cond2(yes)->op3->e2
cond2(no)->e2
```

## 示例

为了验证上述分析内容，对源码增加一些打印，便于我们了解实际的运行机制。

测试代码如下：

```python
import threading


class _RLock:
    def __init__(self):
        self._lock = threading.Lock()
        self._owner = 0
        self._count = 0

    def acquired(self, block=True, timeout=-1):
        me = threading.get_ident()
        if self._owner == me:
            self._count += 1
            print(f"acquired count = {self._count}")
            return 1

        rc = self._lock.acquire(block, timeout)
        if rc:
            self._count = 1
            print(f"acquired count = {self._count}")
            self._owner = me
        return rc

    def release(self):
        me = threading.get_ident()
        if self._owner != me:
            raise RuntimeError("can not release the un-acquired the lock")
        self._count = count = self._count - 1
        print(f"release count = {self._count}")
        if not count:
            self._owner = None
            self._lock.release()


def worker(lock):
    print("====非可重入锁所在线程释放RLock内部锁====")
    pid = threading.current_thread()
    print(f"current pid = {pid}")
    try:
        lock.release()
    except Exception as e:
        print(str(e))
        print("====非可重入锁所在线程释放RLock内部锁失败====")
    print("waiting......")
    lock.acquired()

    print(f"RLock所在线程已释放所有锁，worker线程获取RLock内部锁成功")


if __name__ == '__main__':
    print("生成一个R锁对象")
    rlock = RLock()
    print("主线程获取RLock")
    z = rlock.acquired()
    print(f"current pid = {threading.current_thread()}")
    print("创建并启动一个worker线程")
    t = threading.Thread(target=worker, args=(rlock,))
    t.start()
    print("主线程开始获取RLock锁")
    for i in range(5):
        rlock.acquired()
    print("主线程释放RLock锁")
    for i in range(6):
        rlock.release()
    print("主线程释放完所有的RLock锁")
```

测试代码使用两个线程，一个主线程，一个是 worker 线程。
运行结果如下：（实际结果可能有所出入，但是不影响实际请求流程）

```
生成一个R锁对象
主线程获取RLock
acquired count = 1
current pid = <_MainThread(MainThread, started 140736157979584)>
创建并启动一个worker线程
====非可重入锁所在线程释放RLock内部锁====
主线程开始获取RLock锁
acquired count = 2
acquired count = 3
acquired count = 4
acquired count = 5
acquired count = 6
主线程释放RLock锁
release count = 5
release count = 4
release count = 3
release count = 2
current pid = <Thread(Thread-1, started 123145446993920)>
can not release the un-acquired the lock
====非可重入锁所在线程释放RLock内部锁失败====
waiting......
release count = 1
release count = 0
主线程释放完所有的RLock锁
acquired count = 1
RLock所在线程已释放所有锁，worker线程获取RLock内部锁成功
```
