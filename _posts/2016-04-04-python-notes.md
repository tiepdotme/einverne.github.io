---
layout: post
title: "Python 容易混淆的知识点"
tagline: ""
description: ""
category: 学习笔记
tags: [python, class, inheritance, ]
last_updated:
---

## String Bytes and Unicode in Python
例子

    # Python 2

    >>> print type("Hello World!")
    <type 'str'>
    # this is a byte string

    >>>print type(u"Hello World!")
    <type 'unicode'>
    # this is a Unicode string

Python 3

    # Python 3

    >>> print(type("Hello World!"))
    <class 'str'>
    # this is a Unicode string

    >>> print(type(b"Hello World!"))
    <class 'bytes'>
    # this is a byte string

总而言之，Python 2 中字面 str 以 bytes 存储，前缀 u'hello' 表示 unicode 对象，以 unicode 存储。

而 Python 3，字面 str 以 Unicode 存储，前缀 b'hello' 表示获取 bytes 对象，或者 str.encode() 来获取 bytes 对象。

Python 3.x 中有三种 string 对象类型，一种是文本类型，另外两种是二进制数据

- `str` 表示 Unicode 文本 (both 8bit and wider)
- `bytes` 表示二进制内容
- `bytearray`，一种可变的 `bytes` 类型

### encoding vs decoding
类型转换

- encoding 是将 string 根据 encoding name 转变为 raw bytes
- decoding 是将 raw string 根据 encoding name 转变为 string

将 string 转换为 raw bytes

- `str.encode()` 或者 `str(B, encoding)`

将 raw bytes 转变为 str

- `bytes(S, encoding)` 或者 `bytes.decode()`

### Unicode 编码
在处理 Unicode 编码文本，Python 支持

- "\xNN" hex byte value escapes
- "\uNNNN" and "\UNNNNNNNN" Unicode escapes，在 unicode escapes 中，前一种使用 4 个十六进制数字编码 2-byte(16-bit) 字符串，后一种使用 8 个十六进制数字编码 4-byte(32-bit) 文本

### ASCII 编码
在处理 ASCII 编码时

    >>> ord('X')
    88
    >>> chr(88)
    'X'

    >>> S = 'XYZ'
    >>> S.encode('ascii')               # Values 0..127 in 1 byte (7 bits) each
    >>> S.encode('latin-1')             # Values 0..255 in 1 byte (8 bits) each
    S.encode('utf-8')                   # Values 0..127 in 1 byte, 128..2047 in 2, others 3 or 4

### 非 ASCII 编码
在处理非 ASCII 编码时，可能会用到之前提到的 Unicode 编码

    >>> chr(0xc4)                       # 0xC4, 0xE8: characters outside ASCII's range
    >>> S = '\xc4\xe8'                  # Single byte 8-bit hex escapes
    >>> S = '\u00c4\u00e8'              # 16-bit Unicode escapes
    >>> len(S)                          # 2 characters long (not number of bytes!)

### 编解码非 ASCII 编码
如果我们使用 `encode` 来将一个非 ASCII 字符串使用 ASCII 编码转变为 raw bytes ，会抛出错误。

    >>> S = '\u00c4\u00e8'
    >>> S.encode('ascii')
    UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1:
    ordinal not in range(128)

    >>> S.encode('latin-1')             # One byte per character
    b'\xc4\xe8'

    >>> S.encode('utf-8')               # Two bytes per character
    b'\xc3\x84\xc3\xa8'

    >>> len(S.encode('latin-1')))
    2
    >>> len(S.encode('utf-8'))
    4


## 多重继承
在 Python 中如果一个类继承自多个类，这种行为被称为多重继承 multiple inheritance，虽然多重继承非常有用，但是需要注意一些问题，否则会出现不可预见的问题。

在使用多重继承时，如果一个方法从多个超类中继承，先继承的类中的方法会重写后继承类中的方法。（超类顺序为定义 class 时的顺序）。

## property
`@property` 是一个将用户计算的东西伪装成为一个属性。

    from math import pi

    class Circle():
        def __init__(self,r):
            self.r = r

        @property
        def area(self):
            return pi*self.r**2

    c = Circle(2)
    print(c.area)

在这里 `area()` 虽然被定义为一个方法，但是类的实例可以使用点来访问 `area` ，假装是类的一个属性。

## 静态方法和类成员方法
Python 2.4 中，引入了**装饰器**(decorators) 的语法，能够对任何可调用的对象进行包装，既能够用于方法也能够用于函数。使用 `@` 操作符，在方法或函数上将装饰器列出，指定一个或者多个装饰器。多个装饰器在应用时的顺序与指定顺序相反。

    class Date(object):

        def __init__(self, day=0, month=0, year=0):
            self.day = day
            self.month = month
            self.year = year

        @classmethod
        def from_string(cls, date_as_string):
            day, month, year = map(int, date_as_string.split('-'))
            date1 = cls(day, month, year)
            return date1

        @staticmethod
        def is_date_valid(date_as_string):
            day, month, year = map(int, date_as_string.split('-'))
            return day <= 31 and month <= 12 and year <= 3999

    date2 = Date.from_string('11-09-2012')
    is_date = Date.is_date_valid('11-09-2012')

`classmethod` 必须有一个指向 class object 的 reference 作为第一参数，而 staticmethod 则不需要。

### Class Method
C++ 有重载的功能，但是 Python 缺乏重载的机制，所以就有了 `classmethod`，可以想象成另外一个构造函数

    @classmethod
    def from_string(cls, date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        date1 = cls(day, month, year)
        return date1

    date2 = Date.from_string('11-09-2012')

`cls` 是一个持有 class self 的对象，但是不是 class 的一个实例。如果继承了 `Date` 类，所有的子类都会拥有 `from_string` 方法。

### Static method
对于这个例子，`is_date_valid` 方法不需要关心类内部的其他变量和方法，但是这个 valid 方法和 Date 相关，可以定义为静态方法。和其他语言中的静态方法可以一起理解。

## getattr setattr
拦截 intercept 对象的所有特性访问是可能的，然后有一些魔法方法。比如 `__getattr__` 和 `_setattr__`

- `__getattribute__(self.name)` 当特性 name 被访问时自动被调用，只能在新式类中使用
- `__getattr__(self.name)` 当特性 name 被访问且对象没有相应的特性时被自动调用
- `__setattr__(self.name.value)` 当试图给特性 name 赋值时被自动调用
- `__delattr__(self.name)` 当试图删除特性 name 时被自动调用

这里是一些区别

- `__getattribute__` 无条件被调用，任何对象的属性访问时都会隐式调用，比如 `t.__dict__` 其实是执行了 `t.__getattribute__("__dict__")` ，所以如果我们在重载`__getattribute__`中又调用`__dict__`的话，会无限递归，用 object 来避免，即 object.__getattribute__(self, name)
- `__getattr__` 只有 `__getattribute__` 找不到的时候才会调用 `__getattr__`

举例

    class Attr(object):

        def __init__(self, name):
            self.name = name

        def __getattribute__(self, item):
            print("__getattribute__ " + item)
            return object.__getattribute__(self, item)

        def __getattr__(self, item):
            print("__getattr__ " + item)

        def __setattr__(self, key, value):
            print("__setattr__ key %s, value %s" % (key, value))
            object.__setattr__(self, key, value)


    if __name__ == '__main__':
        attr = Attr('wendy')
        print("main " + attr.name)
        attr.name = 'nancy'

在这个例子中，输出结果

    __setattr__ key name, value wendy
    __getattribute__ name
    main wendy
    __setattr__ key name, value nancy


## 迭代器

在 Python 中，很多对象都是可以通过 for 语句来直接遍历的，例如 list、string、dict 等等，这些对象都可以被称为可迭代对象。迭代器对象要求支持迭代器协议的对象，在 Python 中，支持迭代器协议就是实现对象的`__iter__()` 和 `next()` 方法。其中`__iter__()` 方法返回迭代器对象本身；`next()` 方法返回容器的下一个元素，在结尾时引发 StopIteration 异常。

    class PowTwo:
        """Class to implement an iterator
        of powers of two"""

        def __init__(self, max=0):
            self.max = max

        def __iter__(self):
            self.n = 0
            return self

        def __next__(self):
            if self.n <= self.max:
                result = 2 ** self.n
                self.n += 1
                return result
            else:
                raise StopIteration


    p = PowTwo(5)
    for i in p:
        print(i)


## 生成器
生成器通过生成器函数产生，生成器函数可以通过常规的 def 语句来定义，但是不用 return 返回，而是用 yield 一次返回一个结果，在每个结果之间挂起和继续它们的状态，来自动实现迭代协议。

在理解生成器之前，首先要先理解上面提到的迭代器，使用迭代器能够非常轻松的遍历列表等等中的内容，虽然迭代器很好用，但是迭代器会将内容全都放到内存中，所以一旦遇到特别庞大的列表，可能就会遇到问题。

所以有了生成器，生成器是一种特殊的迭代器，只能够遍历一次。

    >>> mygenerator = (x*x for x in range(3))
    >>> for i in mygenerator:
    ...    print(i)

这里 mygenerator 需要使用 `()`。

然后 `yield` 关键字，就像 `return` ，但是表示方法会返回一个生成器。

    >>> def createGenerator():
    ...    mylist = range(3)
    ...    for i in mylist:
    ...        yield i*i
    ...
    >>> mygenerator = createGenerator() # create a generator
    >>> print(mygenerator) # mygenerator is an object!
    <generator object createGenerator at 0xb7555c34>
    >>> for i in mygenerator:
    ...     print(i)
    0
    1
    4

返回 `yield` 的方法在方法被调用的时候并不会执行，只有在使用 `for` 来遍历该生成器时，才会执行。执行的顺序是，方法每 yield 一次就返回一次，等待 for 中执行完毕，继续执行生成器方法中的下一次 yield，然后返回 for 中，然后反复执行直到生成器没有 yield 任何内容。

## 模块
告诉编译器去哪里找，一种方法就是编辑 `sys.path` 另外一种方法就是使用 `PYTHONPATH` 环境变量

    Python 3.6.1 (default, Apr 23 2017, 12:32:16)
    [GCC 5.4.0 20160609] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import sys
    >>> print(sys.path)
    ['', '/home/einverne/.pyenv/versions/3.6.1/lib/python36.zip', '/home/einverne/.pyenv/versions/3.6.1/lib/python3.6', '/home/einverne/.pyenv/versions/3.6.1/lib/python3.6/lib-dynload', '/home/einverne/.pyenv/versions/3.6.1/lib/python3.6/site-packages']

对于不同的操作系统 `PYTHONPATH` 配置可能有些差异，在类 Unix 系统下

    export PYTHONPATH=$PYTHONPATH:~/pypath:~/py1

### 包
为组织模块，可以分组为包 package, 为了让 Python 将其作为包对待，必须包含一个 `__init__.py` 的文件。

假如构造了如下的目录结构

    ├── douban
    │   ├── album.py
    │   ├── celebrity.py
    │   ├── __init__.py

定义了 douban 目录，那么该文件夹下的 `__init__.py` 就是 douban 包（模块）的代码，其他两个 `album.py` 和 `celebrity.py` 分别是 album 和 celebrity 模块。

如果单纯的引入

    import douban

那么 `__init__` 模块的内容是可用的，但是其他两个模块是不可用的。

    import douban.album

那么这个时候 album 模块也是可用的。但这时候只能通过全名 `douban.album.` 来使用。

    from douban import celebrity

这时候 celebrity 模块可用，可以通过短名来使用，比如 `celebrity.xxx`

模块中定义 `__all__` = [] 向外导出可用方法。在别人应用该模块时，打印 `__all__` 内容就能够清晰看到模块对外提供的方法。

### 标准库
sys 模块能够轻松访问 Python 解释器联系的变量和函数

函数或变量            | 描述
----------------------|---------------------
argv                | 命令行参数，包括脚本名字
exit([arg])         | 退出，可选参数给定返回值或错误
modules             | 映射模块名字到载入模块的字典
path                | 查找模块所在目录的目录名列表
platform            | 平台
stdin               | 标准输入流
stdout              | 标准输出流
stderr              | 标准错误流

os 模块提供了访问操作系统服务的功能。

`os.path`子模块包含了检查构造删除目录和文件的函数。

time 模块能够实现，获取当前时间，操作时间和日期，从字符串读取时间以及格式化时间为字符串等等功能。

函数                    | 描述
------------------------|------------------------
asctime([tuple])        | 将时间元组转换为字符串
localtime([secs])       | 将秒转换为日期元组，本地时间
mktime(tuple)           | 将时间元组转换为本地时间
sleep(secs)             | 休眠
strptime(string[, format])  | 将字符串解析为时间元组
time()                  | 当前时间，秒

random 模块包括返回随机数的函数，可以用于模拟或者任何产生随机数的程序。

## reference

- Python 基础教程
- <https://pythonguidecn.readthedocs.io/zh/latest/>