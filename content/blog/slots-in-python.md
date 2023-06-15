---
title: __slots__ in Python
date: 2017-04-14
tags: ["til", "python"]
---

今天在公司内部交流会上聊到了我们Python后端代码中的一个小问题：在一小段代码中，我们遍历了所有的"item"，然后选了一个ID，把每个"item"都塞进了一个dict里面。
先不论这段代码的具体功能，光从实现来说当初的设计咋一看没有太大的问题，但是现在，我们客户数据库里面动不动就能翻出上千个"item"时，这段代码带来了不小的内存开销，而我们能怎么改善这个情况呢？

<!--more-->

首先，class是解决不了问题的：默认下，每个class都有一个内建的dict，把attribute names对应到具体的数值，再加上一些别的metadata，class占用的内存会比dict更加多。

```python
class Rectangle:
    def __init__(self, top, bottom, left, right):
      self.top = top
      self.bottom = bottom
      self.left = left
      self.right = right

r = Rectangle(1, 2, 3, 4)
print(r.__dict__)
```

`collections.namedtuple`是大部分人的第一反应，然而和tuple一样，namedtuple的attributes是immutable的。
这个差异会给codebase带来不小的改动，而我们都认为这不是一个合适的时间投资。

```python
import collections
Rectangle = collections.namedtuple('Rectangle', ['top', 'bottom', 'left', 'right'])
r = Rectangle(1, 2, 3, 4) # AttributeError: can't set attribute
```

在我们的代码里面，所有的attributes都是已知的，数量不多，而且我们的代码并没有增减attributes，所以我们最后用了`__slots__`这个方案。
在class内声明`__slots__`后，该类将不会自动生成`__dict__`。
我们也能对已知的attributes重新赋值。
需要注意的是，为了使用`__slots__`，我们必须让`Rectangle`继承`object`。
任何`Rectangle`的派生类都必须重新声明`__slots__`，否则派生类还是会内建`__dict__`。

```python
class Rectangle(object):
    __slots__ = ('top','bottom','left', 'right')
    def __init__(self, top, bottom, left, right):
      self.top = top
      self.bottom = bottom
      self.left = left
      self.right = right

r = Rectangle(1, 2, 3, 4)
print(r.__dict__) # AttributeError: 'Rectangle' object has no attribute '__dict__'
print(r.__slots__) # ('top', 'bottom', 'left', 'right')
print(r.top) # = 1
r.top = 3
```

上面的方案存在的最后一个问题是：原代码块内使用的是dict的接口，而我们的类并不兼容这些接口(JS大法好)。
所以我们还必须继承`MutableMapping`，并且实现其所需的方法。
这样的话，我们就可以在基本不改动原代码块的情况下，达到我们减少内存使用的目的。

```python
class Rectangle(MutableMapping):
    __slots__ = ('top','bottom','left', 'right')
    def __init__(self, top, bottom, left, right):
      self.top = top
      self.bottom = bottom
      self.left = left
      self.right = right

    # implements MutableMapping
    ...
```

说回来，在Python里面获取instances所占用的内存的方法并不是很trivial，等我多做一些研究后再写一篇归纳。
