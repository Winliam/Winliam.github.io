---
title: python笔记
categories:
  - - 工作技能
    - python笔记
date: 2019-07-22 17:13:22
tags:
  - python
  - split
  - 函数装饰器
  - 深拷贝
  - 浅拷贝
math: true
---
### python的zip函数
输入若干个Iterable对象，将这些Iterable对象对应位置的元素打包为一个tuple，然后将所有tuple作为一个zip对象返回。

当传入的Iterable对象长度不一致时取最短的。
```
a = [[1,2],[3,4],[5,6]]
b = zip(*a)
c = tuple(b)

>>> a = ((1, 3, 5), (2, 4, 6))
```

### python string的split方法
```
a = ", my, name, is, ming"
b = a.split(',')

>>> b = ['',' my',' name',' is',' ming' ]
```
即，将原string按照字符串分割，不包含分隔符，返回一个列表。

### pyhton 函数装饰器
对被修饰的函数增加一层封装，被修饰的函数仍然正常被调用，被执行，但是增加了一些“私货”，比如打log。这么做的好处是在不修改原函数代码以及调用原函数的代码的前提下改变其行为。
```
import logging

def use_logging(func):
	def wrapper(*args, **kwargs):
		logging.warn("%s is running" % func.__name__)
		return func(*args)
	return wrapper

@use_logging
def foo():
	print("i am foo")

@use_logging
def bar():
	print("i am bar")

foo()
bar()

>>> i am foo
>>> i am bar
>>> WARNING:root:foo is running
>>> WARNING:root:bar is running
```

### python 赋值&浅拷贝&深拷贝的区别
1. 对于不可变对象，三者同质，都只增加原对象的引用计数
2. 对于可变对象
  - 赋值，仍然是引用计数
  - 浅拷贝，新瓶装旧酒，瓶是新的，酒还是原对象元素的计数引用
  - 深拷贝，实质意义上的拷贝，啥都重新搞一份

### python __init__.py文件的作用
一个包含了__init__.py目录可以被识别为一个包，而目录内的.py文件是这个包的模块。一个包也是一个模块，反之则不一定。

在import一个包时，实际上是在执行这个包的__init__.py中的代码。看一个例子，

```
.
├── main.py
└── mypackage
    ├── __init__.py
    ├── subpackage_1
    │   ├── test11.py
    │   └── test12.py
    └── subpackage_2
        ├── __init__.py
        ├── test21.py
        └── test22.py
```
在main.py中`import mypackage`时，python后台程序会按照以下顺序搜索`mypackage`
1. 当前的工作目录；
1. PYTHONPATH（环境变量）中的每一个目录；
1. Python 默认的安装目录。

本例中，在当前工作目录下即可找到`mypackage`，然后会执行其下__init__.py中的代码。

对于mypackage的__init__.py，值得注意的是：
```
import subpackage_1			# 错误，因为在main的当前目录找不到subpackage_1
import subpackage_2			# 错误
import mypackage.subpackage_1		# 正确，先找到mypackage再找到subpackage_1
import mypackage.subpackage_2		# 正确
from . import subpackage_1		# 正确，在from指定的目录可以找到subpackage_1
from . import subpackage_2		# 正确
from mypackage import subpackage_1 	# 正确，同上
from mypackage import subpackage_2

from mypackage import test11			# 错误
from mypackage.subpackage_1 import test11	# 正确
from .subpackage_1 import test11		# 正确，语义同上
from mypackage import test21			
from mypackage.subpackage_2 import test21
from .subpackage_2 import test21
```

### `itertools`库
```
v1 = [(i, j) for i, j in itertools.product(range(2), repeat=2)]
v2 = [(i, j) for i, j in itertools.product(range(2), repeat=3)]

>>> v1 = [(0, 0), (0, 1), (1, 0), (1, 1)]
>>> v2 = [(0, 0, 0), (0, 0, 1), (0, 1, 0), (0, 1, 1), (1, 0, 0), (1, 0, 1), (1, 1, 0), (1, 1, 1)]
```
