---
title: "shell环境变量与shell变量"
layout: markdown
comments: true
tags: shell变量 shell环境变量 setenv export
---

UNIX中有两种变量，环境变量和shell变量，参见[UNIX中的变量](http://www.ee.surrey.ac.uk/Teaching/Unix/unix8.html)。 shell变量只对当前的进程有效，它被用来设置短期的工作环境。 环境变量则是在login登陆后就一直存在，会对整个会话起作用。环境变量一般大写，shell变量一般小写。

- env,printenv 打印当前的环境变量
- set 打印当前的shell变量和环境变量（就是所有变量）


#### 设置一个shell变量
```bash
foo="hello world"
#或者
set foo="hello world"

set | grep foo
env | grep foo
```
上面的例子，你会发现shell变量中有foo，但是环境变量中没有foo。

而shell变量只对当前进程有效，__对子进程无效、对子进程无效、对子进程无效__；所以可能你以为有效的Makefile或者工具却不能拿到你的变量值。

#### 设置一个环境变量

设置环境变量最好的办法就是，在`~/.bashrc`中`export foo="hello world"`导出变量到整个session 环境中。

export命令就是将变量导入到环境变量中。

```bash
export foo
env | grep foo
```
在上面例子基础上，export变量，变量就被导出到了环境中。

参考文章[笨办法学linux中文版](https://wizardforcel.gitbooks.io/llthw/content/ex5.html)
