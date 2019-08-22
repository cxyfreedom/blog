---
title: Python 中 Redis 连接池原理
date: 2019-04-25T18:33:45+08:00
description: ""
tags: ["Python", "redis", "连接池", "原理"]
categories: ["Python"]
---
通常情况下，我们进行 `redis` 操作时，会创建一个连接，并基于这个连接进行操作。操作完成后，释放该连接。但是当并发较高时，频繁的创建和释放会对性能造成影响。连接池的原理是通过预先创建多个连接。进行获取已经创建好的连接进行操作，操作完成之后，不会进行释放，可以用于后续其他redis的操作。

<!--more-->
## 原理

### 连接池的使用

```python
rds = redis.StrictRedis(connection_pool=redis.ConnectionPool(host=host, port=port, password=password, db=rdb))
```

### redis.ConnectionPool 实例化

```python
def __init__(self, connection_class=Connection, max_connections=None,
             **connection_kwargs):
    """
    Create a connection pool. If max_connections is set, then this
    object raises redis.ConnectionError when the pool's limit is reached.

    By default, TCP connections are created connection_class is specified.
    Use redis.UnixDomainSocketConnection for unix sockets.

    Any additional keyword arguments are passed to the constructor of
    connection_class.
    """
    max_connections = max_connections or 2 ** 31
    if not isinstance(max_connections, (int, long)) or max_connections < 0:
        raise ValueError('"max_connections" must be a positive integer')

    self.connection_class = connection_class
    self.connection_kwargs = connection_kwargs
    self.max_connections = max_connections

    self.reset()
```
此时并未做任何的 redis 连接，仅仅设置了最大连接数、连接参数和连接类

### StrictRedis 实例化

```python
def __init__(self, ...connection_pool=None...):
        if not connection_pool:
            ...
            connection_pool = ConnectionPool(**kwargs)
        self.connection_pool = connection_pool
```
即使不创建连接池, 它也会自己创建。
```python
def execute_command(self, *args, **options):
    "Execute a command and return a parsed response"
    pool = self.connection_pool
    command_name = args[0]
    connection = pool.get_connection(command_name, **options)
    try:
        connection.send_command(*args)
        return self.parse_response(connection, command_name, **options)
    except (ConnectionError, TimeoutError) as e:
        connection.disconnect()
        if not connection.retry_on_timeout and isinstance(e, TimeoutError):
            raise
        connection.send_command(*args)
        return self.parse_response(connection, command_name, **options)
    finally:
        pool.release(connection)
```
当执行 Redis 命令时，每个命令内部会调用 `execute_command` 进行操作。
```python
# 调用的是 ConnectionPool 的 get_connection
connection = pool.get_connection(command_name, **options)
```
该行创建了连接。
```python
def get_connection(self, command_name, *keys, **options):
    "Get a connection from the pool"
    self._checkpid()
    try:
        connection = self._available_connections.pop()
    except IndexError:
        connection = self.make_connection()
    self._in_use_connections.add(connection)
    return connection
```
从代码中可以看出，如果有可用的连接，获取可用的连接。如果没有，创建一个。
```python
def make_connection(self):
    "Create a new connection"
    if self._created_connections >= self.max_connections:
        raise ConnectionError("Too many connections")
    self._created_connections += 1
    return self.connection_class(**self.connection_kwargs)
```
在 ConnectionPool 的实例中, 有两个 list, 分别是 `_available_connections` 和 `_in_use_connections`, 分别表示**可用的连接集合**和**正在使用的连接集合**, 在上面的`get_connection`中, 我们可以看到获取连接的过程是：

1. 从可用连接集合尝试获取连接,
2. 如果获取不到, 重新创建连接
3. 将获取到的连接添加到正在使用的连接集合

```python
def release(self, connection):
    "Releases the connection back to the pool"
    self._checkpid()
    if connection.pid != self.pid:
        return
    self._in_use_connections.remove(connection)
    self._available_connections.append(connection)
```
命令执行完成后，将使用完的可用连接放回连接列表里面连接池对象调用 `release` 方法, 将连接从`_in_use_connections` 放回 `_available_connections`, 这样后续的连接获取就能再次使用这个连接了。