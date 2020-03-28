---
title: "Tornado 模板引擎原理解析"
date: 2020-03-28T21:53:30+08:00
lastmod: 2020-03-28T21:53:30+08:00
draft: false
keywords: ["Tornado", "模板引擎", "Template", "Python", "解析", "原理"]
description: ""
tags: ["Tornado", "模板引擎", "Python", "原理"]
categories: ["Python"]
---

<!--more-->
> 本文内容是基于 Tornado 4.5.2 和 Python 3.6.4 版本进行解析

## 模板引擎的用法

在 Tornado 中，有两种方式来渲染模板并输出。一种是渲染字符串的方式，另一种是通过渲染模板文件。

### 渲染字符串

比如有下列代码：
```python
from tornado import template


t = template.Template("<html>{{ value }}</html>")
print(t.generate(value='demo'))
```
执行上述代码输出：
```python
b'<html>demo</html>'
```

### 渲染模板文件

代码如下：
```python
from tornado import template


loader = template.Loader("./")
t = loader.load("index.html")  # 加载模板
print(t.generate(value="demo"))
```
`index.html` 内容为：
```html
<html>{{ value }}</html>
```
执行上述代码输出：
```python
b'<html>demo</html>'
```

## 模板引擎的渲染

当调用 `t = template.Template("<html>{{ value }}</html>")` 初始化时，比较核心的逻辑是：
```python
def __init__(self, template_string, name="<string>", loader=None,
                 compress_whitespace=_UNSET, autoescape=_UNSET,
                 whitespace=None):
    # 省略部分
    self.namespace = loader.namespace if loader else {}
        reader = _TemplateReader(name, escape.native_str(template_string),
                                 whitespace)
        self.file = _File(self, _parse(reader, self))
        self.code = self._generate_python(loader)
        self.loader = loader
        try:
            # Under python2.5, the fake filename used here must match
            # the module name used in __name__ below.
            # The dont_inherit flag prevents template.py's future imports
            # from being applied to the generated code.
            self.compiled = compile(
                escape.to_unicode(self.code),
                "%s.generated.py" % self.name.replace('.', '_'),
                "exec", dont_inherit=True)
        except Exception:
            formatted_code = _format_code(self.code).rstrip()
            app_log.error("%s code:\n%s", self.name, formatted_code)
            raise
```

也就是说，Tornado 在加载字符串或者模板文件后，首先进行模板内容的语法解析，然后将其编译成 Python 代码。当我们渲染模板时，就嵌入对应的数据得到最终的输出。

* `_TemplateReader` 记录模板内容读取到哪个位置
* `_parse` 函数就是语法解析的具体实现
* `_generate_python` 会调用 `_File` 类中的 `generate` 方法会生成对应需要执行的 Python 代码
* `compile` 方法将代码进行编译

对于字符串 `<html>{{ value }}</html`，会被编译成如下代码：
```python
def _tt_execute():  # <string>:0
    _tt_buffer = []  # <string>:0
    _tt_append = _tt_buffer.append  # <string>:0
    _tt_append(b'<html>')  # <string>:1
    _tt_tmp = value  # <string>:1
    if isinstance(_tt_tmp, _tt_string_types): _tt_tmp = _tt_utf8(_tt_tmp)  # <string>:1
    else: _tt_tmp = _tt_utf8(str(_tt_tmp))  # <string>:1
    _tt_tmp = _tt_utf8(xhtml_escape(_tt_tmp))  # <string>:1
    _tt_append(_tt_tmp)  # <string>:1
    _tt_append(b'</html>')  # <string>:1
    return _tt_utf8('').join(_tt_buffer)  # <string>:0
```

我们将模板文件改进一下。分为 base.html 和 index.html。base.html 文件如下：
```html
# base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Title{% end %}</title>
</head>
<body>
<ul>
    {% for post in posts %}
        {% block post %}
            <li>{{ post['content'] }}</li>
        {% end %}
    {% end %}
</ul>
</body>
</html>
```
index.html 文件如下：
```html
{% extends "base.html" %}
{% block title %}Blog{% end %}
{% block post %}
  <li>{{ escape(post['content']) }}</li>
{% end %}
```

然后我们执行 Python 代码：
```python
from tornado import template


loader = template.Loader("./")
t = loader.load("index.html")
posts = [{"content": "a"}, {"content": "b"}]
print(t.generate(posts=posts))
print(loader.templates['base.html'].code)  # 获取 base.html 编译后的代码
```
最终输出：
```python
b'<!DOCTYPE html>\n<html lang="en">\n<head>\n<meta charset="UTF-8">\n<title>Blog</title>\n</head>\n<body>\n<ul>\n\n\n<li>a</li>\n\n\n\n<li>b</li>\n\n\n</ul>\n</body>\n</html>'
```
上述在编译后 `base.html` 的代码：
```python
def _tt_execute():  # base.html:0
    _tt_buffer = []  # base.html:0
    _tt_append = _tt_buffer.append  # base.html:0
    _tt_append(b'<!DOCTYPE html>\n<html lang="en">\n<head>\n<meta charset="UTF-8">\n<title>')  # base.html:5
    _tt_append(b'Title')  # base.html:5 (via base.html:5)
    _tt_append(b'</title>\n</head>\n<body>\n<ul>\n')  # base.html:9
    for post in posts:  # base.html:9
        _tt_append(b'\n')  # base.html:10
        _tt_append(b'\n<li>')  # base.html:11 (via base.html:10)
        _tt_tmp = post['content']  # base.html:11 (via base.html:10)
        if isinstance(_tt_tmp, _tt_string_types): _tt_tmp = _tt_utf8(_tt_tmp)  # base.html:11 (via base.html:10)
        else: _tt_tmp = _tt_utf8(str(_tt_tmp))  # base.html:11 (via base.html:10)
        _tt_tmp = _tt_utf8(xhtml_escape(_tt_tmp))  # base.html:11 (via base.html:10)
        _tt_append(_tt_tmp)  # base.html:11 (via base.html:10)
        _tt_append(b'</li>\n')  # base.html:12 (via base.html:10)
        _tt_append(b'\n')  # base.html:13
        pass  # base.html:9
    _tt_append(b'\n</ul>\n</body>\n</html>')  # base.html:16
    return _tt_utf8('').join(_tt_buffer)  # base.html:0
```

我们来分析一下这段编译后的 Python 代码：

* `_tt_buffer` 就是存放每一部分渲染后的片段的列表
* `_tt_append` 其实就是 `_tt_buffer` 这个列表的 append 方法
* 纯文本内容会直接 append 到 `_tt_buffer` 中，比如 `_tt_append(b'</title>\n</head>\n<body>\n<ul>\n')`
* 如果是类似 `{% for %}` 包裹的语句，那么会直接按照 Python 中的逻辑执行再进行处理，然后依次 append 到 `_tt_buffer` 中
* 如果是 `{% %}` 包裹的语句，会根据不同的指令做对应的处理
* 最后将每一部分渲染后的片段拼接起来

用一句话来说，Tornado 模板引擎渲染输出的方法就是：**根据语法解析，把模板内容分割为一个个的块，这个块被称为 Chunk 或者 Node，然后把每个 Chunk 转换为对应的 Python 中的代码，最终拼接成渲染后的模板**。输出的逻辑被封装在 `_tt_execute` 这个函数中，执行 `generate` 方法也就是调用这个函数并嵌入数据得到输出结果。

## 语法解析

根据上述内容我们可以知道，无论是渲染字符串还是渲染模板，首先都会对内容进行语法解析。语法解析的处理就是用过 `_parse` 这个函数来进行的。
```python
def _parse(reader, template, in_block=None, in_loop=None):
    body = _ChunkList([])
    while True:
        # Find next template directive
        curly = 0
        while True:
            curly = reader.find("{", curly)
            if curly == -1 or curly + 1 == reader.remaining():
                # EOF
                if in_block:
                    reader.raise_parse_error(
                        "Missing {%% end %%} block for %s" % in_block)
                body.chunks.append(_Text(reader.consume(), reader.line,
                                         reader.whitespace))
                return body
            # If the first curly brace is not the start of a special token,
            # start searching from the character after it
            if reader[curly + 1] not in ("{", "%", "#"):
                curly += 1
                continue
            # When there are more than 2 curlies in a row, use the
            # innermost ones.  This is useful when generating languages
            # like latex where curlies are also meaningful
            if (curly + 2 < reader.remaining() and
                    reader[curly + 1] == '{' and reader[curly + 2] == '{'):
                curly += 1
                continue
            break

        # Append any text before the special token
        if curly > 0:
            cons = reader.consume(curly)
            body.chunks.append(_Text(cons, reader.line,
                                     reader.whitespace))

        start_brace = reader.consume(2)
        line = reader.line

        # Template directives may be escaped as "{{!" or "{%!".
        # In this case output the braces and consume the "!".
        # This is especially useful in conjunction with jquery templates,
        # which also use double braces.
        if reader.remaining() and reader[0] == "!":
            reader.consume(1)
            body.chunks.append(_Text(start_brace, line,
                                     reader.whitespace))
            continue

        # Comment
        if start_brace == "{#":
            end = reader.find("#}")
            if end == -1:
                reader.raise_parse_error("Missing end comment #}")
            contents = reader.consume(end).strip()
            reader.consume(2)
            continue

        # Expression
        if start_brace == "{{":
            end = reader.find("}}")
            if end == -1:
                reader.raise_parse_error("Missing end expression }}")
            contents = reader.consume(end).strip()
            reader.consume(2)
            if not contents:
                reader.raise_parse_error("Empty expression")
            body.chunks.append(_Expression(contents, line))
            continue

        # Block
        assert start_brace == "{%", start_brace
        end = reader.find("%}")
        if end == -1:
            reader.raise_parse_error("Missing end block %}")
        contents = reader.consume(end).strip()
        reader.consume(2)
        if not contents:
            reader.raise_parse_error("Empty block tag ({% %})")

        operator, space, suffix = contents.partition(" ")
        suffix = suffix.strip()

        # Intermediate ("else", "elif", etc) blocks
        intermediate_blocks = {
            "else": set(["if", "for", "while", "try"]),
            "elif": set(["if"]),
            "except": set(["try"]),
            "finally": set(["try"]),
        }
        allowed_parents = intermediate_blocks.get(operator)
        if allowed_parents is not None:
            if not in_block:
                reader.raise_parse_error("%s outside %s block" %
                                         (operator, allowed_parents))
            if in_block not in allowed_parents:
                reader.raise_parse_error(
                    "%s block cannot be attached to %s block" %
                    (operator, in_block))
            body.chunks.append(_IntermediateControlBlock(contents, line))
            continue

        # End tag
        elif operator == "end":
            if not in_block:
                reader.raise_parse_error("Extra {% end %} block")
            return body

        elif operator in ("extends", "include", "set", "import", "from",
                          "comment", "autoescape", "whitespace", "raw",
                          "module"):
            if operator == "comment":
                continue
            if operator == "extends":
                suffix = suffix.strip('"').strip("'")
                if not suffix:
                    reader.raise_parse_error("extends missing file path")
                block = _ExtendsBlock(suffix)
            elif operator in ("import", "from"):
                if not suffix:
                    reader.raise_parse_error("import missing statement")
                block = _Statement(contents, line)
            elif operator == "include":
                suffix = suffix.strip('"').strip("'")
                if not suffix:
                    reader.raise_parse_error("include missing file path")
                block = _IncludeBlock(suffix, reader, line)
            elif operator == "set":
                if not suffix:
                    reader.raise_parse_error("set missing statement")
                block = _Statement(suffix, line)
            elif operator == "autoescape":
                fn = suffix.strip()
                if fn == "None":
                    fn = None
                template.autoescape = fn
                continue
            elif operator == "whitespace":
                mode = suffix.strip()
                # Validate the selected mode
                filter_whitespace(mode, '')
                reader.whitespace = mode
                continue
            elif operator == "raw":
                block = _Expression(suffix, line, raw=True)
            elif operator == "module":
                block = _Module(suffix, line)
            body.chunks.append(block)
            continue

        elif operator in ("apply", "block", "try", "if", "for", "while"):
            # parse inner body recursively
            if operator in ("for", "while"):
                block_body = _parse(reader, template, operator, operator)
            elif operator == "apply":
                # apply creates a nested function so syntactically it's not
                # in the loop.
                block_body = _parse(reader, template, operator, None)
            else:
                block_body = _parse(reader, template, operator, in_loop)

            if operator == "apply":
                if not suffix:
                    reader.raise_parse_error("apply missing method name")
                block = _ApplyBlock(suffix, line, block_body)
            elif operator == "block":
                if not suffix:
                    reader.raise_parse_error("block missing name")
                block = _NamedBlock(suffix, block_body, template, line)
            else:
                block = _ControlBlock(contents, line, block_body)
            body.chunks.append(block)
            continue

        elif operator in ("break", "continue"):
            if not in_loop:
                reader.raise_parse_error("%s outside %s block" %
                                         (operator, set(["for", "while"])))
            body.chunks.append(_Statement(contents, line))
            continue

        else:
            reader.raise_parse_error("unknown operator: %r" % operator)
```

`_parse` 函数的第一个参数 `reader` 是 `_TemplateReader` 类的实例，主要是用来记录模板文件中所在的位置。

这个函数的实现逻辑主要分为以下几个步骤：

1. 首先定义了一个 `_ChunkList` 的实例，用来保存解析的内容。这个类的作用后面再详细说
2. 从模板内容的最开始位置开始寻找指令
3. 如果找到合法的指令，先将这个指令之前的文本转为 `_Text` 类型的节点，然后 append 到 body 中。再判断当前指令的类型，将指令转为对应类型的节点，再 append 到 body 中
4. 如果没有找到合法的指令，就一直找到末尾，并将内容转为 `_Text` 类型的节点，然后 append 到 body 中

其中 `reader.consume(count=None)` 方法有点类似滑动窗口，通过 `reader.find()` 方法找到字符串的位置，然后滑动多少位置到匹配的末尾。

## Chunk / Node

可以说 Chunk，或者说是 Node 是 Tornado 模板引擎实现的关键。经过 `_parse` 的语法解析，将模板内容拆分成了多个 Chunk，最后将多个 Chunk 进行整合输出。理解了每个 Chunk 在做什么，就等于理解了实现的原理。

在 `_parse` 中定义过一个 `_ChunkList`，这个类继承了 `_Node` 类。
```python
class _Node(object):
    def each_child(self):
        return ()

    def generate(self, writer):
        raise NotImplementedError()

    def find_named_blocks(self, loader, named_blocks):
        for child in self.each_child():
            child.find_named_blocks(loader, named_blocks)


class _ChunkList(_Node):
    def __init__(self, chunks):
        self.chunks = chunks

    def generate(self, writer):
        for chunk in self.chunks:
            chunk.generate(writer)

    def each_child(self):
        return self.chunks
```

`_ChunkList` 就是所有节点的集合，每个节点类型都是 `_Node` 的子类，`_Node` 是所有节点类型的基类。

`_Node` 类中有三个方法，作用分别是L
* `each_child`：返回该节点的所有子节点
* `generate`：生成节点对应的 Python 代码
* `find_named_blocks`：用来寻找节点中 block 的命名块

### 节点类型

Tornado 模板引擎中，定义的节点类型主要可以分为以下几种：

* 文本节点
* 注释节点
* 表达式节点
* 块节点

**文本节点**

通常来说，文本节点就是普通的不会被特殊渲染的字符串，字符串编译后的结果还是该字符串。

**注释节点**

注释节点是以 `{#` 开头，以 `#}` 结束。注释节点编译后不会输出到最终结果中。

**表达式节点**

表达式节点是以 `{{` 开头，以 `}}` 结束，中间一个字符串。这个字符串可以是变量名，也可以是一个复杂的表达式。比如 `{{ name }}`，`{{ add(1, 2) }}`。

**块节点**

块节点是以 `{%` 开头，以 `%}` 结束，中间的第一个字符串是**指令**，不同的指令有不同的处理方法，不同的块节点也可以进行组合。

| 指令 | 使用方式 | 映射 |
| :-: | :-: | :-: |
| apply | {% apply *function* %}...{% end %} | 调用这个function，这个代码块中内容就是这个输入 |
| autoescape | {% autoescape *function* %} | 设置转义格式 |
| block | {% block *name* %}...{% end %} | 自定义命名块，并替换 extends 中父模板中对应的命名块 |
| comment | {% extends *filename* %} | 注释 |
| extends | {% extends *filename* %} | 继承父模板 |
| for | {% for *var* in *expr* %}...{% end %} | Python 中的 for 语句 |
| from | {% from *x* import *y* %} | Python 中的 import 语句 |
| if | {% if *condition* %}...{% elif *condition* %}...{% else %}...{% end %} | Python 中的 if 语句 |
| import | {% import *module* %} | Python 中的 import 语句 |
| include | {% include *filename* %} | 直接包含另一个文件（能够获取文件中的本地变量） |
| module | {% module *expr* %} | 渲染一个 tornado.web.UIModule |
| raw | {% raw *expr* %} | 直接输出表达式结果，不转义 |
| set | {% set *x* = *y* %} | 设置一个本地变量 |
| try | {% try %}...{% except %}...{% else %}...{% finally %}...{% end %} | Python 中的 try 语句 |
| while | {% while *condition* %}... {% end %} | Python 中的 while 语句 |
| whitespace | {% whitespace *mode* %} | 空白处理 |

`_Node` 有如下这些子类：

| _Node 子类 | 类型 |
| :-: | :-: |
| _File | 整个模板 |
| _ChunkList | 多个节点的集合 |
| _NameBlock | 命名块，即 block 指令 |
| _ExtendsBlock | 继承，即 extends 指令 |
| _IncludeBlock | 导入，即 include 指令 |
| _ApplyBlock | 即 apply 指令 |
| _ControlBlock | 控制块，即 if 指令 |
| _IntermediateControlBlock | 对应 else, elif, except, finally 之类的指令 |
| _Statement | 对应 import, from, set, break 之类的指令 |
| _Expression | 对应 raw 指令 |
| _Module | 对应 module 指令 |
| _Text | 对应文本节点 |

例如 `_Text` 子类，它的实现如下：
```python
class _Text(_Node):
    def __init__(self, value, line, whitespace):
        self.value = value
        self.line = line
        self.whitespace = whitespace

    def generate(self, writer):
        value = self.value

        # Compress whitespace if requested, with a crude heuristic to avoid
        # altering preformatted whitespace.
        if "<pre>" not in value:
            value = filter_whitespace(self.whitespace, value)

        if value:
            writer.write_line('_tt_append(%r)' % escape.utf8(value), self.line)
```

它只实现了 `generate` 方法，因为文本仅仅是单纯的字符串，所以没有子节点或者命名块。在函数中，就处理了一下空白。

> pre 元素中通常保留空格和换行符，所以不处理

其它的子类实现原理也是类似，大多就是 `generate` 内部的实现不同。

## 生成 Python 代码

生成 Python 代码主要就是调用 `_generate_python` 函数，我们先来看一下它的实现：
```python
class Template(object):
    # 省略其它代码
    def _generate_python(self, loader):
        buffer = StringIO()
        try:
            # named_blocks maps from names to _NamedBlock objects
            named_blocks = {}
            
            # 获取父模板的 _FILE
            ancestors = self._get_ancestors(loader)
            # 翻转让父模板在最前，最终我们是要通过父模板来渲染
            ancestors.reverse()
            # 找出当前模板中名称相同的块，用子块替换父模板中同名的块，并存放到 named_blocks 中
            for ancestor in ancestors:
                ancestor.find_named_blocks(loader, named_blocks)
            writer = _CodeWriter(buffer, named_blocks, loader,
                                 ancestors[0].template)
            # 根据父模板生成
            ancestors[0].generate(writer)
            return buffer.getvalue()
        finally:
            buffer.close()

    def _get_ancestors(self, loader):
        ancestors = [self.file]
        for chunk in self.file.body.chunks:
            if isinstance(chunk, _ExtendsBlock):
                if not loader:
                    raise ParseError("{% extends %} block found, but no "
                                     "template loader")
                template = loader.load(chunk.name, self.name)
                ancestors.extend(template._get_ancestors(loader))
        return ancestors
```
方法内，首先会调用 `self._get_ancestors(loader)`。这个函数的作用是获取当前模板的父模板，如果当前模板的 `_FILE` 中有 `_ExtendsBlock`，就说明它有父模板，就通过 `loader.load(chunk.name, self.name)` 加载父模板，内部会调用 `_create_template` 来创建父模板的 Template 实例，递归调用父模板的 `_get_ancestors` 方法得到父模板的 `_FILE` 节点集合。

假设没有多重继承，那么就会获得 `[当前模板的 _FILE, 父模板的 _FILE]`。然后翻转，让第一个是父模板。

`ancestor.find_named_blocks(loader, named_blocks)` 内部就是通过 `named_blocks[self.name] = self` 将块替换成当前模板的块。此时我们遍历是从父模板开始，最后到子模板，所以最后使用的就是当前模板的块。

最终，把父模板的内容拷贝到 buffer 中并返回。

## 执行并输出

编译成 Python 代码后，就可以通过 `Template.generate()` 方法最终渲染输出。

我们来看一下 `generate` 函数的代码：
```python
def generate(self, **kwargs):
    """Generate this template with the given arguments."""
    namespace = {
        "escape": escape.xhtml_escape,
        "xhtml_escape": escape.xhtml_escape,
        "url_escape": escape.url_escape,
        "json_encode": escape.json_encode,
        "squeeze": escape.squeeze,
        "linkify": escape.linkify,
        "datetime": datetime,
        "_tt_utf8": escape.utf8,  # for internal use
        "_tt_string_types": (unicode_type, bytes),
        # __name__ and __loader__ allow the traceback mechanism to find
        # the generated source code.
        "__name__": self.name.replace('.', '_'),
        "__loader__": ObjectDict(get_source=lambda name: self.code),
    }
    namespace.update(self.namespace)
    namespace.update(kwargs)
    exec_in(self.compiled, namespace)
    execute = namespace["_tt_execute"]
    # Clear the traceback module's cache of source data now that
    # we've generated a new template (mainly for this module's
    # unittests, where different tests reuse the same name).
    linecache.clearcache()
    return execute()
```

Tornado 自定义了一个 namespace，并把之前各种节点用于模板渲染的参数插入到这个 namespace 中。

`exec_in(self.compiled, namespace)` 包装了 Python 中的 exec 函数，为了兼容 Python2 和 Python3。（Python2 是作为语句，Python3 作为函数）
```python
def exec_in(code, glob, loc=None):
    # type: (Any, Dict[str, Any], Optional[Mapping[str, Any]]) -> Any
    if isinstance(code, basestring_type):
        # exec(string) inherits the caller's future imports; compile
        # the string first to prevent that.
        code = compile(code, '<string>', 'exec', dont_inherit=True)
    exec(code, glob, loc)
```

exec 用于执行存储在字符串或者文件中的 Python 代码。它的语法格式是：
```python
exec(object[, globals[, locals]])
```

* object：必选参数，表示需要被指定的 Python 代码。如果是字符串，会先被解析成 Python 代码后，在执行；如果是 code 对象，就直接执行
* globals：可选参数，用于存放全局变量，如果提供，必须是一个 dict
* locals：可选参数，用于存放局部变量，如果提供，可以是任何 mapping；如果忽略，就等同于 globals

示例：
```python
x = 10
expr = """
z = 30
sum = x + y + z
print(sum)
"""
def func():
    y = 20
    exec(expr)  # 60
    exec(expr, {'x': 1, 'y': 2})  # 33
    exec(expr, {'x': 1, 'y': 2}, {'y': 3, 'z': 4})  # 34
    
func()
```

最终获取 `namespace["_tt_execute"]` 中的 Python 代码并执行。

在实际的 Web 开发中，我们大多数情况并不会直接使用 Template 对象来渲染输出，而是会通过 RequestHandler 中的 `render` 方法和 `render_string` 方法。

`render_string` 方法是根据模板的名称和上下文参数来对模板进行渲染。它的 namespace 是通过方法内部的 `get_template_namespace()` 方法来获取的。

```python
# tornado/web.py
def get_template_namespace(self):
    """Returns a dictionary to be used as the default template namespace.

    May be overridden by subclasses to add or modify values.

    The results of this method will be combined with additional
    defaults in the `tornado.template` module and keyword arguments
    to `render` or `render_string`.
    """
    namespace = dict(
        handler=self,
        request=self.request,
        current_user=self.current_user,
        locale=self.locale,
        _=self.locale.translate,
        pgettext=self.locale.pgettext,
        static_url=self.static_url,
        xsrf_form_html=self.xsrf_form_html,
        reverse_url=self.reverse_url
    )
    namespace.update(self.ui)
    return namespace
```

## 总结

本文先描述了 Tornado 模板引擎的使用方法，到一步步地解析它的实现原理，它的逻辑实现并不复杂，Chunk 的设计，以及在渲染模板时，通过树的结构，利用 DFS 来查找块和替换，都是我们常见的经典设计方法和数据结构算法。从理解到最后整理成文，是一次很好的阅读源码实现的过程。

如果文中有错误，也可以评论或者邮件等方式和我交流，谢谢。