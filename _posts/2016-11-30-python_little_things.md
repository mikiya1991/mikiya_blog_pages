---
title: python little things
layout: markdown
comments: true
---

# python中令人费解的小细节
---------------------------------

## 在python中获得帮助

在linux环境中，我们可以简单的通过`man`、`info`、`--help`的方式来获得关于指令的详细的信息甚至获得到example。实在找不到，再求助谷歌。

对于这些指令，我认为其实没有必要逐一的记住。因为一个是很辛苦，第二辛苦记住的指令可能日常中并没有那么的常用，可能日常90%使用的指令也就那么几十个而已，完全没有必要去记住那么多。**真正重要的是记住，指令的用途和原理**, 然后只要在使用的时候`man`一下就好了。

因此对于python，我觉得也是如此，语言的设计思想给你的启发是最重要的，反而语法和类库的具体的步骤没有那么的重要。使用的时候再查询就好了。

> 授人以鱼不如授人以渔

### python的文档

关于python，首先推荐[廖雪峰大神的博客](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013747381369301852037f35874be2b85aa318aad57bda000)了，是我找到的关于python最好的文档了。

另外就是**官方的文档**。在windows上的idle编辑器中可以直接通过窗口上的help按钮获得帮助文档。

一个重要的办法就是**看书和查找官方的手册**。这是我们系统有体系地学习的最主要的方法，一本好书起到的效果是事半功倍的。例如，学习unix编程最好的方法，莫过于去看APUE，经典的权威书籍，是少走弯路最好的办法。并且书是作者智慧的结晶，一个厉害的作者写出来的书，你会看到他智慧和视野、而不是仅仅代码和教条。

<br/>

### python中的help

如何方便的在语言中查询我们想要的信息，是编程中很重要的技巧，对于类库来讲更是这样。

在python，目前获得具体类、属性的方法有下面三种。

1. **dir** 函数

    `dir(obj)`会以一个列表列出当前对象的，所有方法和属性。以此我们可以对它有基本的了解，甚至可以使用代码来查找是否有我们想要的方法。

2. **help** 函数

    `help(obj)`会生成一个关于类库的手册，类似linux中的`man`命令，是python中最主要的获得帮助的方式。

        >>> help(str)
        Help on class str in module __builtin__:
        class str(basestring)
        |  str(object='') -> string
        |  
        |  Return a nice string representation of the object.
        |  If the argument is a string, the return value is the same object.
        |  
        |  Method resolution order:
        |      str
        |      basestring
        |      object
        |  
        |  Methods defined here:
        |  
        |  __add__(...)
        |      x.__add__(y) <==> x+y
        |  
        |  __contains__(...)
        |      x.__contains__(y) <==> y in x
        |  
        |  __eq__(...)
        |      x.__eq__(y) <==> x==y
        |  
        |  __format__(...)
        :

3. **\_\_doc\_\_** 属性

    **\_\_doc\_\_** 属性会返回方法代码前的注释信息，他是python开发者编写注释生成帮助的手段。 我猜想，其实可能help函数就是通过类的doc属性生成的文档。

    另外doc属性和help都可以直接对对象的方法使用。

        >>> str.split.__doc__
        'S.split([sep [,maxsplit]]) -> list of strings\n\nReturn a list of the words in the string S, using sep as the\ndelimiter string.  If maxsplit is given, at most maxsplit\nsplits are done. If sep is not specified or is None, any\nwhitespace string is a separator and empty strings are removed\nfrom the result.'

-------------------------------------

## 一些python魔术变量的作用

python中有很多类似`__x__`形式的变量，它们被称为魔术变量，是一些有特殊用途的变量。

### 程序中的环境变量

我们打开python命令行，然后输入`dir()`，结果如下。

    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__']

dir获得了当前环境（对象）的所有属性和方法。我们发现什么都没有做的python命令行中，其实python已经替我们加载了这么多东西了。使用print打印这些变量的值，如下所示。

    >>> print __doc__
    None
    >>> print __name__
    __main__
    >>> print __package__
    None
    >>> print __builtins__
    <module '__builtin__' (built-in)>
    >>> type(__name__)
    <type 'str'>

由此可大概知道，当前环境其实可以理解为在一个对象内。该对象的`__name__`属性值是一个字符的`__main__`。而且python还给我们加载了内建的`builtin`模块，使我们可以用内建的对象的方法。

该对象还有个`__package__`属性，表示该对象所属于的软件包。  
因为对象的`__name__`属性值是`__main__`，所以python程序默认从这里开始运行。  
`__doc__`是该对象的帮助文档，这里为空。

我们还可以使用help命令来查看`__builtins__`的属性。

    Help on built-in module __builtin__:
    NAME
      __builtin__ - Built-in functions, exceptions, and other objects.
    FILE
      (built-in)
    DESCRIPTION
      Noteworthy: None is the `nil' object; Ellipsis represents `...' in slices.
    CLASSES
      object
        basestring
            str
            unicode
        buffer
        bytearray
        classmethod
        complex
        dict
        enumerate


#### import之后

那么，如果在当前环境import一个类库之后会发生什么呢？

    >>> import numpy
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__', 'numpy']

从上面结果可知，当import一个类库之后，其就会被加载到当前环境中了。

<br/>

### import过程以及相关的魔术变量

本节参考自[python官方对module的解释](https://docs.python.org/2/tutorial/modules.html)

#### module

在python中所有的py文件都是一个模块（module）,每个模块都有属性`__name__`，一般就是该文件的文件名。

因此如果在同一目录下存在一个fibo.py的文件，就可以认为是一个名为fibo的模块。可以直接使用import加载，并且模块名及它的`__name__`属性就是文件名。

    >>> import fibo
    >>> fibo.fib(1000)
    1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987
    >>> fibo.fib2(100)
    [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
    >>> fibo.__name__
    'fibo'

**让模块以script运行**  
例如直接在命令行运行时，该模块的`__name__`属性就是字符串`__main__`。

#### package
python中的包（pkg）是一种由模块组成的更大的软件仓库，可以使用类似`foo.bar.baz`的方式来访问大的软件包中某一个模块，而`foo`就是一个很多子模块组成的包。在python中，这也是一种名称空间的用法。

例如如下的一个文件目录结构中，只要`sound`目录下有`__init__.py`文件，它就是一个pkg。我们及可以使用`sound.formats.wavread`来加载wavread.py文件。

    sound/                          Top-level package
      __init__.py               Initialize the sound package
      formats/                  Subpackage for file format conversions
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  Subpackage for filters
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...

#### module的接口
本节参考自文章[用 \_\_all\_\_ 暴露接口](http://python-china.org/t/725)

`__all__`变量，是模块加载时起作用的。

    import os
    import sys
    __all__ = ["process_xxx"]  # 排除了 `os` 和 `sys`
    def process_xxx():
      pass  # omit

不像 Ruby 或者 Java，Python 没有语言原生的可见性控制，而是靠一套需要大家自觉遵守的”约定“下工作。比如下划线开头的应该对外部不可见。同样，`__all__` 也是对于模块公开接口的一种约定，比起下划线，`__all__` 提供了暴露接口用的”白名单“。一些不以下划线开头的变量（比如从其他地方 import 到当前模块的成员）可以同样被排除出去。

#### package中的`__path__`变量

软件包支持一个更特殊的属性，`__path__`变量（模块不支持）。

该变量可以写在package的 `__init__.py`中，是一个路径名的列表。我们可以在package顶层目录中的`__init__.py`文件中给path赋值，这样pkg就可以包含其它路径中的子模块。这个特性跟环境变量中的path差不多。

参考文章[The import system](https://docs.python.org/3/reference/import.html)。

---------------------------------------

## python import系统原理

## 类中的魔法变量
python类中的魔术方法一般都是用来进行运算符重载的，另外还有一些对象初始化，getter、setter，容器的特殊操作。

其实就是重载了运算符，len、print、for等调用的默认方法。

具体请参考文章[Python 魔术方法指南](http://pycoders-weekly-chinese.readthedocs.io/en/latest/issue6/a-guide-to-pythons-magic-methods.html#id22)，或者官方文档。

## 最后

本文都是我自己对python中的一些疑惑，查找并综合得出的一些对python语言的理解。

其实在我探究问题的途中，靠谱的谷歌已经多次将我引荐给了python官方文档中的《language reference》篇。原来一切都是因为我 “刚学会走就想跑”的后果。就在我质疑python的时候，我就必须更加深入地去了解它。因此想要更深的了解python，去看更深的官方文档吧。（https://docs.python.org/3/reference/index.html）
