---
title: "c语言中的static、const关键字"
layout: markdown
comments: true
tags: static const c
---

例如局部的字符：
```c
int func() {
	static const char * const str = "abcd";
}
```

定义了一个局部静态的字符串`str`。那这几个static，const是起了什么作用呢？

## const

`char *str`是一个名叫str的char \*（char指针）。 `*`之前的char代表指针指向的内容的类型，之后的str代表指针的名字。

- 如果const在`*`之前，则const修饰的是指针指向的类型
- const在`*`之后，则修饰的是指针本身

所以const在`*`之前就表示str指向的 *内容不可变* 。即“abcd”不可以被修改。  
const在`*`之后便是str *指针本身不可以改变*，指针指向不能变，即是str不能指向别的字符串。

参考[C语言中const关键字的用法](https://blog.csdn.net/xingjiarong/article/details/47282255)

__const在\*之前，可以防止指针指向的内容被改变；const在\*之后，防止指针指向别的地方。__

	Tip： `const char *str` 与 `char const *str`相同，const只看相对星号的位置。

## static

表示数据放在data段，程序一开始就会初始化。而不是在堆栈中，是一直存在。

static全局变量，就是全局变量。  
static局部变量，也是一直存在，但是外部不可见。
