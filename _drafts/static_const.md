---
title: c语言中的static、const关键字
layout: markdown
comments: true
keyword: static, const, c语言
---

# c语言中的static和const关键字

## const关键字

const表示变量是常量

局部的字符数组为：
static const char * const strs[] =

{"a", "b", "c"}
static表示其放在data段，一直都在； 一开始就会初始化。
第一const，作用于char，const char， 即指针指向的值是常量。
第二个const，作用于 strs[]， 表示strs[]是常量，即strs[]本身不可变。
const 作用编译期间检查，用于防止错误。

此处指针应该为常量，指向字符也应该是常量。同时应该在一开始就存在。所以应该是
static const char * const strs[] = ...
