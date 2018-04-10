---
title: unix信号
comments: true
layout: markdown
---

unix信号——初步
====================
--------------------
_本文章来自APUE第十章学习、总结_

## 1. 概念
信号其实是UNIX的一种IPC（进程间通信）的手段。
是非常简单但是方便的进程间通信。

其实信号的使用场景无非就是在bash中控制进程的运行：
- 用户发送sigterm/int信号给进程，告诉进程终止（ctrl + c）
- 子进程结束的时候，系统会自动发送sigchld给父进程，从而告诉父进程同步
- 定时器sigalrm，定时告诉进程事件
- 进程的运行错误，段错误sigsegv，非法指令sigill，告诉进程自己出错了
- 然后就是不太常用的作业管理，前后台进程的管理 sighup ...


### 1.1 大致的信号的列表

<table class="table table-bordered">
	<tr>
		<th>信号标识</th>
		<th>信号说明</th>
	</tr>
	<tr><td>SIGALRM </td><td>定时器信号。</td></tr>
	<tr><td>SIGFPE </td><td>算数异常，除以0、浮点溢出。</td></tr>
	<tr><td>SIGFREEZE </td><td>系统睡眠，通知应用保存信息。</td></tr>
	<tr><td>SIGHUP </td><td>终端接口检测到连接断开。</td></tr>
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

### 终端作业管理

这一节不太重要，如果不想看可以直接略过。

在终端环境下，用户按键产生的信号，都是终端驱动产生的，及内核产生。终端中前后台程序的区别是，终端运行的程序一般都是前台程序，此时终端一般不能输入其他指令了，只能与程序交互。

但是如果运行命令时在其后加上`&`符号，进程就会切换到后台执行，参见[Shell 前后台进程切换](https://cnbin.github.io/blog/2015/06/15/shell-qian-hou-tai-jin-cheng-qie-huan/)。

切换到后台执行：`Ctrl+z`，挂起前台进程；`bg %[number]`,将其放到后台执行。  
前后台进程有部分指令控制：
- `jobs -l` 
- `fg %n` 
- `bg %n`
参见上面的链接。

---------------------------------

## 2. 最简单的信号处理方式 --- Signal函数

    #include <signal.h>
    typedef void (* pSigalFunc)(int);
    pSignalFunc signal(int signo, pSignalFunc);

- signo: 信号id。
- func： 自定义信号处理的函数（可以是预定义的一些值）
    - 忽略： SIG_IGN
    - 默认： SIG_DFL
    - 捕捉： 自定义的处理函数。返回值void, 参数是当前出发的信号id。
- 返回值：之前信号处理程序指向的指针。失败返回SIG_ERR。

这个是最早的，也是最方便的信号处理方式。

对于一个信号，如果不自定义处理方式，unix会有一个默认的信号处理的行为。

- sigint和sigterm就是停止运行
- sigchld就是忽略
- sigsegv等等错误就是结束运行

如果你要处理信号，就是用signal函数，注册信号处理函数。

*但是signal函数其实有很多的缺点，在稳定性要求的应用中需要注意使用*


