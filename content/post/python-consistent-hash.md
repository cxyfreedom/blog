---
title: 使用 HashRing 实现 Python 下的一致性哈希算法
date: 2019-04-29T14:43:31+08:00
description: ""
tags: ["Python", "一致性哈希"]
categories: ["Python"]
---
当我们需要实现服务器负载均衡的时候，能够实现负载均衡的算法有很多种，包括：轮询算法（Round Robin）、哈希算法（Hash）、最少连接算法（Least Connection）、响应速度算法（Response Time）、加权法（Weighted）等等。
<!--more-->
## 场景

比较常见的应用场景时：有 N 台服务器提供缓存服务，需要对服务器进行负载均衡，能够将请求均分到每台服务器上，即每台服务器接收 1/N 个请求。

## 取模 Hash

根据上述应用场景，最简单的方案就是取模 Hash。假设集群有 N 台服务器，对于用户的请求进行编号，假设某个请求编号为 M，那么就讲这个请求通过取模后发送到对应编号的机器。
```
机器编号 = M % N
```
比如我们现在有以下地址的集群，我们对它们分别进行编号：
```
0 10.10.1.1
1 10.10.2.2
2 10.10.3.3
3 10.10.4.4
```
客户端有 100 个请求需要发送，我们可以设计一个规则让这些请求都携带一个编号，比如 1-100，那么：
```sh
1 % 4 = 1 --> 10.10.2.2
2 % 4 = 2 --> 10.10.3.3
3 % 4 = 3 --> 10.10.4.4
4 % 4 = 0 --> 10.10.1.1
...
```
这样，我们就能够将客户端请求均分到每台服务器上。我们用代码模拟类似：
```python
# -*- coding: utf-8 -*-

content = """In computer science, consistent hashing is a special kind of 
hashing such that when a hash table is resized, only K/n keys need to be 
remapped on average, where K is the number of keys, and n is the number of 
slots. In contrast, in most traditional hash tables, a change in the number 
of array slots causes nearly all keys to be remapped because the mapping 
between the keys and the slots is defined by a modular operation."""

servers = [
    "10.10.1.1",
    "10.10.2.2",
    "10.10.3.3",
    "10.10.4.4",
]


class NormalHash:
    def __init__(self, nodes=None):
        self.nodes = nodes if nodes else []
        self.count = len(nodes) if nodes else 0

    def get_node(self, value):
        if value < 0:
            return
        return self.nodes[value % self.count]


def load_balance():
    nh = NormalHash(servers)
    words = content.split()

    database = {s: [] for s in servers}

    for idx in range(len(words)):
        node = nh.get_node(idx)
        database[node].append(words[idx])

    print(database)


if __name__ == '__main__':
    load_balance()
```
但是取模 Hash 存在一个明显的缺点。如果机器后续需要扩容，比如现在增加了一台服务器节点:
```
4 10.10.5.5
```
那么按照取模的规则：
```sh
1 % 5 = 1 --> 10.10.2.2
2 % 5 = 2 --> 10.10.3.3
3 % 5 = 3 --> 10.10.4.4
4 % 5 = 4 --> 10.10.1.1 # 节点和之前不一致
```
对于新的数据来说，没有影响，但是如果要获取旧的数据，就会发生不一致，因为现在的请求所对应的机器编号和之前可能不一致了。这个时候，我们就需要进行数据迁移。如果一台服务器挂掉，那么就有 (N-1)/N 的服务器的缓存数据需要重新计算存储；如果新增一台机器，会有 N/(N+1) 的服务器的缓存数据需要重新计算存储。

取模 Hash 对于业务比较小的场景来说，比如只有几十台机器来说，完全能够支撑服务需要。而且方案很简单，理解和维护都很方便。如果真的需要扩容集群，最好按照整数倍进行扩容，否则数据迁移的成本太大。

## 一致性 Hash

我们通过图示来理解一致性 Hash。

{{% figure class="center" src="/img/consistent_hash1.jpg" alt="consistent_hash1" %}}

我们设计一个环，假设这个环由 2^32 - 1 个点组成。也就是说 [0, 2^32) 上任意一个点都能够在环上找到。然后我们使用一个算法，比如 md5，把我们集群中的服务器以 IP 地址作为 key，然后根据算法得到其值。这个值对应环上的一点，以及其所对应的数据区间。
```sh
10.10.1.1 --> 1000 --> (50000, 1000]
10.10.2.2 --> 5000 --> (1000, 5000]
10.10.3.3 --> 20000 --> (5000, 20000]
10.10.4.4 --> 50000 --> (20000, 50000]
```
图中粉色的点代表根据某种算法计算得出在环上对应的位置，箭头则代表这个点存储的位置。

对于扩容情况，一致性 Hash 也能够解决这种情况。

{{% figure class="center" src="/img/consistent_hash2.jpg" alt="consistent_hash2" %}}

假设现在增加了 `10.10.5.5` 这台服务器，那么重新计算：
```sh
10.10.1.1 --> 1000 --> (50000, 1000]
10.10.2.2 --> 5000 --> (1000, 5000]
10.10.3.3 --> 20000 --> (5000, 20000]
10.10.5.5 --> 30000 --> (20000, 30000] # 新增节点
10.10.4.4 --> 50000 --> (30000, 50000]
```
此时，受影响的数据只是 (20000, 30000) 这个范围内的数据，只需要迁移这一部分的数据。同样的如果挂了一台服务器，受影响的也只是这台服务器范围的数据点。

一致性 Hash 能够解决热点分布的问题，对于扩容和缩容也能够低成本的进行。但是一致性 Hash 在小规模集群中很容易出现数据热点分布不均匀的问题。当机器数量少的时候，hash 出来很有可能每个节点的范围大小差异很大。如果本身是均匀分布的，在加入一个新的节点，就可能让数据分布变得不均匀。

{{% figure class="center" src="/img/consistent_hash3.jpg" alt="consistent_hash3" %}}

为了解决平衡性问题，引入了**虚拟节点**。

### 虚拟节点

虚拟节点是以实际的节点在 Hash 空间的复制品（replica）。即对每一个服务节点计算哈希，然后每个计算结果位置都放置一个此服务节点。具体做法可以在服务器 IP 或者主机名的后面增加编号，比如 `#`。
```sh
10.10.1.1#1
10.10.1.1#2
10.10.1.1#3
...
```
对于数据定位算法仍旧不变，只是多了一步虚拟节点到实际节点的映射。例如定位到 `10.10.1.1#1`、`10.10.1.1#2` 和 `10.10.1.1#3` 这三个虚拟节点的数据，都认为定位到 `10.10.1.1` 上面。这样就能够解决数据不均匀的问题。

在实际应用中，通常将虚拟节点数设置为 32 甚至更大，这样即使服务节点很少，也能够做到相对均匀的数据分布。
```python
# -*- coding: utf-8 -*-
import hashlib

content = """In computer science, consistent hashing is a special kind of 
hashing such that when a hash table is resized, only K/n keys need to be 
remapped on average, where K is the number of keys, and n is the number of 
slots. In contrast, in most traditional hash tables, a change in the number 
of array slots causes nearly all keys to be remapped because the mapping 
between the keys and the slots is defined by a modular operation."""

servers = [
    "10.10.1.1",
    "10.10.2.2",
    "10.10.3.3",
    "10.10.4.4",
]


class HashRing:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas
        self.ring = dict()
        self._sorted_keys = []

        if nodes:
            for node in nodes:
                self.add_node(node)

    def add_node(self, node):
        """
        Adds a `node` to the hash ring (including a number of replicas)
        """
        for i in range(self.replicas):
            virtual_node = f"{node}#{i}"
            key = self.gen_key(virtual_node)
            self.ring[key] = node
            self._sorted_keys.append(key)
            # print(f"{virtual_node} --> {key} --> {node}")

        self._sorted_keys.sort()
        # print([self.ring[key] for key in self._sorted_keys])

    def remove_node(self, node):
        """
        Removes `node` from the hash ring and its replicas
        """
        for i in range(self.replicas):
            key = self.gen_key(f"{node}#{i}")
            del self.ring[key]
            self._sorted_keys.remove(key)

    def get_node(self, string_key):
        """
        Given a string key a corresponding node in the hash ring is returned.

        If the hash ring is empty, `None` is returned.
        """
        return self.get_node_pos(string_key)[0]

    def get_node_pos(self, string_key):
        """
        Given a string key a corresponding node in the hash ring is returned
        along with it's position in the ring.

        If the hash ring is empty, (`None`, `None`) is returned.
        """
        if not self.ring:
            return None, None

        key = self.gen_key(string_key)
        nodes = self._sorted_keys
        for i in range(len(nodes)):
            node = nodes[i]
            if key < node:
                return self.ring[node], i

        # 如果key > node，那么让这些key落在第一个node上就形成了闭环
        return self.ring[nodes[0]], 0

    def gen_key(self, string_key):
        """
        Given a string key it returns a long value, this long value represents
        a place on the hash ring
        """
        m = hashlib.md5()
        m.update(string_key.encode('utf-8'))
        return m.hexdigest()


def consistent_hash(replicas):
    hr = HashRing(servers, replicas)
    words = content.split()

    database = {s: [] for s in servers}

    for w in words:
        database[hr.get_node(w)].append(w)

    # print(f"words={len(words)}\n")

    for node, result in database.items():
        print(f"{node}={len(result)}\nresult={result}")


if __name__ == '__main__':
    consistent_hash(3)
```

## 参考

* [一致性hash在分布式系统中的应用](http://www.firefoxbug.com/index.php/archives/2791/)
* [面试必备：什么是一致性Hash算法？](https://zhuanlan.zhihu.com/p/34985026)
