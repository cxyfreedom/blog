---
title: Python 中给文件加锁——fcntl
date: 2019-01-08T15:02:07+08:00
description: ""
tags: ["Python", "fcntl"]
categories: ["Python"]
---
在 Python 中，如果想要给一个文件加锁，可以使用 `fcntl` 模块。其中有 `flock` 和 `lockf` 两种不同的方式。首先我们先了解一下在 Linux 中的文件锁。

<!--more-->

## Linux 中的文件锁

在 Linux 中，flock 和 fcntl 都是系统调用，而 lockf 则是库函数。lockf 则是 fcntl 的封装，因此 lockf 和 fcntl 在底层实现是一样的，对文件加锁的效果也是一样的。

首先说一些概念：

* 文件锁：针对整个文件的锁，如 flock。
* 记录锁：针对整个文件和文件部分字节的锁，如 fcntl、lockf。
* 排他锁：也可以称为写锁、独占锁，同一时间只有一个进程可以加锁。
* 共享锁：也可以称为读锁，支持多个进程并发读文件内容，但不可以写。
* 睡眠锁：一般和等待队列同时存在，当无法获取锁的时候会在等待队列中睡眠，直到满足条件被唤醒，如 semaphore、mutex。
* 自旋锁：自旋锁在被持有时，其它进程再申请时将不断”自旋”，不会陷入睡眠，直到持有者释放。为保证性能，自旋锁不应被持有时间过长。
* 劝告锁（建议锁）：不要求进程一定要遵守，是一种约定俗成的规则，某进程持有建议锁的时候，其它进程依然可以强制操作，如 flock、fcntl。
* 强制锁：是内核行为，在系统调用违反约束条件时，内核将直接阻拦，如 fcntl（fcntl也可实现强制锁，但不建议使用）。

### flock

函数：`flock(fd, operation)`

`fd` 是系统调用 open 返回的文件描述符，`operation` 的可选项有：

* LOCK_SH: 共享锁
* LOCK_EX: 排他锁
* LOCK_UN: 解锁
* LOCK_NB: 非阻塞（与上述三种操作一起使用）

flock 和 lockf 的第一个区别是 flock 只能对整个文件进行上锁，而不能对文件的某一部分上锁，lockf 可以对文件的某个区域进行上锁。

第二个区别是 flock 只能产生劝告性锁。flock 可以有共享锁和排他锁，而 lockf 只支持排他锁。

第三个区别主要是在使用 fork/dup 的情况。

第四个区别是 flock 不能在 NFS 文件系统上使用，要在 NFS 上使用需要用 fcntl。

>flock 锁可以递归，即通过 dup 或者 fork 产生的两个 fd，都可以进行加锁而不会死锁。

### lockf 和 fcntl

函数：
```
int fcntl(int fd, int ⌘, ... /* arg */ );
struct flock {
... 
short l_type;/* Type of lock: F_RDLCK, F_WRLCK, F_UNLCK */
short l_whence; /* How to interpret l_start: SEEK_SET, SEEK_CUR, SEEK_END */ 
off_t l_start;   /* Starting offset for lock */ 
off_t l_len;     /* Number of bytes to lock */ 
pid_t l_pid; /* PID of process blocking our lock (F_GETLK only) */ 
...        
};
```

相关的 cmd 有三种：

* F_SETLK: 设置文件锁（非阻塞）
* F_SETLKW: 设置文件锁（阻塞）
* F_GETLK: 获取锁信息，会修改我们传入的 struct flock。

fcntl/lockf 的特性有：

* 上锁可以递归。
* 加读锁（共享锁）必须是读打开的，加写锁（排他锁）文件必须是写打开的。
* 由 fork 产生的子进程不继承父进程所设置的锁。
* 支持强制性锁：对一个特定文件打开其设置组ID位(S_ISGID)，并关闭其组执行位(S_IXGRP)，则对该文件开启了强制性锁机制。再Linux中如果要使用强制性锁，则要在文件系统mount时，使用 `-o mand` 打开该机制。

## 使用 Python 中的 fcntl 模块

加密方式:
```python
import fcntl

fd = open("test")  # flock 生成的是劝告锁，因此进程可以正常打开文件
fcntl.flock(fd, fcntl.LOCK_EX)  # 检测文件是否被加锁。如果已经上了锁，那么这里就会被阻塞
fcntl.flock(fd, fcntl.LOCK_EX | fcntl.LOCK_NB)  # 用 LOCK_NB 就不会被阻塞
fcntl.flock(fd.fcntl.LOCK_UN)  # 对文件进行解锁
```

* 对文件 close 后文件锁会失效
* 进程结束后文件锁会失效

代码示例：
```python
class SingleInstance:

    def __init__(self):
        file_path = os.path.abspath(__file__)
        cur_dir = os.path.dirname(file_path)
        self.lockfile = os.path.normpath(
            cur_dir + '/' + os.path.basename(__file__))

        if sys.platform != 'win32':
            self.fp = open(self.lockfile, 'r')
            try:
                fcntl.flock(self.fp, fcntl.LOCK_EX | fcntl.LOCK_NB)
            except IOError:
                print("Another instance is already running.")
                sys.exit(-1)
        else:
            print("Other platforms are not supported.")
            sys.exit(-1)


if __name__ == '__main__':
    single = SingleInstance()
    while True:
        time.sleep(1)
```