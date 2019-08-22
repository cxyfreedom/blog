---
title: 解析自定义 Logger 后会打印第三方库中 Logger 日志的原因
date: 2018-12-26T15:35:13+08:00
description: ""
tags: ["Python", "logging", "pika"]
categories: ["Python"]
---
在使用第三方库 pika 连接 RabbitMQ 的代码中，自定义了 Logger，并将相关配置写在了配置文件中。但是执行程序发现，pika 库中本身的 Logger 日志也写入到了文件，但我们只希望记录我们自己所写的日志。

<!--more-->

## 问题

> 在使用第三方库 pika 连接 RabbitMQ 的代码中，自定义了 Logger，并将相关配置写在了配置文件中。但是执行程序发现，pika 库中本身的 Logger 日志也写入到了文件，但我们只希望记录我们自己所写的日志。

## 问题解析

从配置文件导入 Logger 相关代码如下：
```python
_config = os.path.normpath(os.path.join(_cur_dir, 'logging.conf'))
logging.config.fileConfig(_config)
```

然后我们进入到 `fileConfig` 这个函数中看。这个函数有一个关键字参数是 `disable_existing_loggers=True`，作用是不打印已经存在的 Logger。在这个函数中，最后调用了 `_install_loggers(cp, handlers, disable_existing_loggers)`，当中的逻辑也包括了设置 Logger 的 disabled 属性。

回到我们源代码，在我们代码中可能是这样写的（修改过）：
```python
from log import debug_logger
import pika
```

在从 `log.py` 导入 `debug_logger` 之前，在该文件中已经执行了 `logging.config.fileConfig(_config)`，但此时由于我们还没有导入 `pika`，因此在 `pika` 中定义的 Logger 并没有将 disabled 属性设置为 True。

## 解决方案

解决这个问题其实也很简单，在我们导入自定义配置文件之前，先导入那些已经存在的 Logger，将其保存在 Manager 中的 `loggerDict`。