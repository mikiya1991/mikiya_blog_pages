---
title: unix信号
comments: true
layout: markdown
---

unix信号——初步
====================
--------------------
_本文章来自APUE第十章学习、总结_

## 概念
信号就是**软件中断**。  
在单线程（多线程中不常用，因为多线程中场景不适合异步操作）的程序中，提供一种**异步处理**机制。  
信号也是一种**进程间通信**的方式。  
但是比起unix socket、消息队列，信号作用更侧重于**作业控制**，而并非数据交换。所谓的作业控制，就是各个进程的运行进度控制，工作进度的统筹等等。  
经常出现的场景是，父进程通过信号控制等待子进程工作的完成(SIGCHLD)、父进程通过信号终止子进程的执行（SIGKILL）。

#### 一些基本知识

- 头文件： linux `<signal.h>`, macOS X `<sys/signal.h>`
- 信号0：
- SIGABRT: 夭折信号，abort函数产生
- SIGALRM: 闹钟信号，alarm函数超时产生
- SIGINT： `Ctrl+C`中断信号，用来停止失控的程序
- SIGSEGV： 硬件异常信号（访问无效内存、除以0）
- kill命令：　用来向一个进程发送信号。（有限制，同属一个父线程或有root权限）

-------------------

## 信号的处理

#### 信号处理的三种方式
1. 忽略信号

    除了`SIGKILL` `SIGSTOP`外的几乎所有信号都可以被忽略。这两个信号不能被忽略，是因为其为**内核及root提供程序终止的方式**。
2. 捕捉信号

    在内核某种信号发生时，调用一个用户提供的函数。  
    典型的信号处理的例子：

    - 捕捉`SIGCHID`信号获得子进程状态改变的通知，然后在异步函数中，使用`waitpid`函数获得子进程的状态信息。

    - 捕捉`SIGTERM`信号获得程序终止的通知，如果程序存在临时文件或其它，在其终止时，`SIGTERM`处理函数中，将其删除。（SIGTERM是终止信号，kill默认发送终止信号，？？注意不能捕捉SIGKILL、SIGSTOP信号）。　　

3. 执行默认动作（大多数的默认动作是终止进程）

    信号的默认动作见APUE{第三版－P251}

#### 各种信号的说明

<table class="table table-bordered">
	<tr>
		<th>信号标识</th>
		<th>信号说明</th>
	</tr>
	<tr><td>SIGALRM </td><td>定时器信号。</td></tr>
	<tr><td>SIGFPE </td><td>算数异常，除以0、浮点溢出。</td></tr>
	<tr><td>SIGFREEZE </td><td>系统睡眠，通知应用保存信息。</td></tr>
	<tr><td>SIGHUP </td><td>？？终端接口检测到连接断开。</td></tr>
	<tr><td>SIGILL </td><td>非法硬件指令。</td></tr>
	<tr><td>SIGIO </td><td>异步IO信号。</td></tr>
	<tr><td>SIGPIPE </td><td>管道读进程终止时写管道，则触发此信号。</td></tr>
	<tr><td>SIGSEGV </td><td>段访问错误</td></tr>
	<tr><td>SIGSYS </td><td>无效的系统调用。</td></tr>
	<tr><td>SIGCHLD </td><td>一个进程终止或停止时，该信号发给父进程（停止也指暂停）。</td></tr>
	<tr><td>SIGCONT </td><td>发送给停止的进程，使其继续。</td></tr>
	<tr><td>SIGSTOP </td><td>作业控制信号，暂停一个进程。<strong>不能被捕捉、忽略</strong>。</td></tr>
	<tr><td>SIGINT </td><td>用户中断，用户按<code>Ctrl+C</code>，终端驱动产生该信号，给前台所有进程。</td></tr>
	<tr><td>SIGQUIT </td><td>用户退出键<code>Ctrl+\</code>，与<code>Ctrl+C</code>效果相同，但产生core文件。</td></tr>
	<tr><td>SIGABRT </td><td>进程调用abort产生此信号。用于异步处理不正常退出。</td></tr>
	<tr><td>SIGTERM </td><td>kill默认产生的终止信号。可以捕获，使程序优雅的退出。</td></tr>
	<tr><td>SIGKILL </td><td><mark>不能捕捉、忽略</mark>，用于内核、root杀死进程。</td></tr>x`
	<tr><td>SIGTSTP </td><td>交互停止信号，<code>Ctrl+Z</code>挂起终端，发送至前台进程组，<strong>前台程序暂停</strong>。</td></tr>
	<tr><td>SIGTTIN </td><td>后台进程组进程试图读前台控制终端时，终端驱动程序产生，通知读。</td></tr>
	<tr><td>SIGTTOU </td><td>后台进程组写控制终端。</td></tr>
</table>

#### 终端作业管理
在终端环境下，用户按键产生的信号，都是终端驱动产生的，及内核产生。终端中前后台程序的区别是，终端运行的程序一般都是前台程序，此时终端一般不能输入了。但是如果运行命令时在其后加上`&`符号，进程就会切换到后台执行，参见[Shell 前后台进程切换](https://cnbin.github.io/blog/2015/06/15/shell-qian-hou-tai-jin-cheng-qie-huan/)。

切换到后台执行：`Ctrl+z`，挂起前台进程；`bg %[number]`,将其放到后台执行。  
前后台进程有部分指令控制 `jobs -l` `fg %n` `bg %n`，参见上面的链接。

---------------------------------

## Signal函数

    #include <signal.h>
    typedef void (* pSigalFunc)(int);
    pSignalFunc signal(int signo, pSignalFunc);

- signo: 信号名。
- func：
    - 忽略： SIG_IGN
    - 默认： SIG_DFL
    - 捕捉： 自定义的处理函数。返回值void, 参数信号值。
- 返回值：之前信号处理程序指向的指针。失败返回SIG_ERR。

在终端中启动程序，以及fork出子程序时的注意点，见APUE{p258}。

### 函数指针
C中函数是一个代码地址， 因此函数指针是指向该地址的一个指针。他们是不同的。

	typedef void (* pfunc)(void); 	//定义一个函数指针类型
	typedef void func(void);		//定义一个函数类型

	pfunc p1;
	func *p2;
	//两者意义相同

	//同时指针可以强制类型转换
	pfunc p3 = (pfunc)-1;
	pfunc p4 = (void (*)(void))-1;
	//两者相同

函数名也是一个函数指针，只不过它是一个**指针常量** 。因为函数就是一个地址，所以函数就是一个函数指针。
可以认为pfunc 和 func等同（调用和赋值中），但是申明时不等同。

	void Func(){
		return;
	}
	p1 = &Func;
	p1 = Func;
	//1合理，2是语法糖

	(*p1)();
	p1();
	//两者都可调用函数，1合理，2是语法糖。

详情参见文章[深入理解C语言函数指针](http://www.cnblogs.com/windlaughing/archive/2013/04/10/3012012.html)。


