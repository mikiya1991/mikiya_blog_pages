---
title: "unix信号"
comments: true
layout: markdown
---

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
	<tr><td>SIGKILL </td><td><mark>不能捕捉、忽略</mark>，用于内核、root杀死进程。</td></tr>
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


## 3. 信号处理中的一些问题

### 3.1 早期unix系统，信号是不可靠的
1. 信号可能会丢失；进程控制信号的能力弱，只能捕捉或者忽略，不能阻塞信号（告诉系统记住信号，然后在需要的时候再处理）
2. 早期，进程进入信号处理函数后，系统总是将信号的处理方式改为默认方式。（于是信号处理程序中总是重新设置信号处理函数，但是这中间存在时间空隙，很可能造成信号的默认处理行为的发生。下面代码的sigint的默认处理会造成程序的退出）

```
sig_int()
{
	/*reestablish handler for next occurrence*/
	signal(SIGINT, sig_int);
	...
}
```

3. 进程无法阻塞信号（让系统记住信号，然后等到进程准备好再处理），操作系统只能选择处理或忽略，这可能造成处理的错误。

```
int sig_int_flag;

int main()
{
	...
	while (sig_int_flag == 0)
		pause();
	...
}

sig_int()
{
	signal(SIGINT, sig_int);
	sig_int_flag = 1;
}

```

这段程序如果在while和pause之间发生一次sigint信号，程序就会永远僵死（pause函数的功能是僵死，直到发生信号时被唤醒）。但是如果程序能在while句时阻塞了信号sigint的处理，就不会发生上面的情况。

### 3.2 信号会中断系统调用
对于用户的代码，信号会中断当前代码指令的执行，跳转到信号处理的代码中，等信号处理部分代码完成退出后，会恢复用户代码的环境然后自动执行。因此对于普通的用户代码，对其几乎没有影响。

但是如果不是用户代码，而是用户当时正在阻塞地调用一个系统调用（比如io操作，waitpid等系统调用）时发生一个信号，信号也会中断其执行。但是系统调用的环境可是无法还原的，因为涉及内核层的调用。

常见的系统调用被中断的结果是（以阻塞read为例）：
- 返回失败，置errno为EINTR。（SystemV派生系统）
- 返回成功，返回的是已经读写的数据量。（BSD派生系统）
- 自动重启被中断的系统调用。（BSD4.2以后的版本会自动重启中断的系统调用，4.3版本则允许关闭自动重启）

对于前两种情况，在涉及阻塞的系统调用时，常见的写法是检测返回值是不是EINTR或者没有达到读写要求，然后反复调用。（此写法其实是为了防止被信号中断）

对于自动重启的应用，自然是不需要检测，但是可能也有坏处，因为有些时候不想自动重启。

同时对于自动重启，各个系统的版本实现相当不同，systemV是不重启，bsd是重启，linux和mac osx在使用signal函数处理中断时是自动重启；而sigaction则是自己定义是不是要重启（通过设置一个标志位SA_RESTART）。所以对于signal函数各个系统的行为是未知的。

### 3.3 信号中断的函数重入问题
在信号处理的函数中很容易发生函数重入的问题。例如：


> 如果程序此时正在malloc操作中，发生一个信号，触发了信号处理函数，而信号的处理函数中又调用了malloc函数。由于malloc函数会内部读写一个链表，所以信号处理函数中如果打断malloc操作，然后重新进入malloc函数很可能造成内部链表的破坏。

对于此问题的解决方式：
将函数分为可重入和不可重入，对于可重入的函数（不存在全局变量处理和同步问题），信号处理函数是安全的；但是对于不可重入的函数，是有风险的，因此在此类函数的处理过程中，要屏蔽掉一些信号的发生。

### 3.4 SIGCHD的处理方式不同
SIGCHD跟SIGCHLD不是一个信号，是系统V特有的信号，他有特有的信号处理方式。

正常的SIGCHLD信号的处理方式中，如果设置为SIG_IGN，调用进程忽略该信号，并且子进程会产生僵死的进程，必须等待主进程使用wait函数来获得其退出状态。

一般程序设计中，为了获取子进程退出状态并且不产生僵死进程，可以捕捉SIGCHLD信号，然后在处理函数中调用wait函数获取进程退出状态。

但是系统V的SIGCHD信号处理方式与其他信号的处理方式不同

- 如果SIGCHD设置为忽略，则不会产生僵死的进程，子进程状态丢弃；如果设置为默认处理方式，则跟正常信号处理方式相同，产生僵死进程，等待主进程wait其状态。
> 系统V这样的处理是为了兼容系统4中的信号SIGCHLD，系统4中的SIGCHLD与BSD和POSIX中定义的行为不同；并且系统4版的sigaction，也可以设置SA_NOCLDWAIT标志来避免产生僵死进程。
- 如果捕捉SIGCHD，则signal函数执行时立即会检查是否有进程准备好被等待（而不是默认的在有子进程状态改变后触发）。
> 这种行为会造成问题，一般老的signal处理中，会立即调用signal函数立即设置信号处理函数（因为老版本的系统会在信号处理函数内将信号的处理行为设置为default），但是如果在SIGCHD处理函数中设置signal，signal会立即检测是否有准备好的进程，然后就会再次出发信号处理函数，不断循环直到堆栈用尽，出错退出。 所以在SIGCHD的处理函数中必须先wait进程，然后再重新设置信号处理函数。

## 4 怎样才是可靠的信号
- 信号应当存在三个状态：产生、处理、未决（产生之后还未被处理）
- 进程可以阻塞住一个信号的处理，使信号保持在未决状态。（进程可以使用sigpending函数阻塞住信号）
- 如果某个信号阻塞过程中，发生了很多次，应当对信号进行排队。（但是当前的unix系统只支持递送信号一次）
- 多个信号同时递送时，信号次序问题。（POSIX.1没有规定，只是建议优先递送与进程状态相关的信号，如sigsegv）
- 每个进程有一个信号屏蔽字，规定当前阻塞的信号集。进程可以调用sigprocmask来检测和更改当前信号屏蔽字。
- 信号集的数据类型sigset_t

## 5 发送信号

```
int kill(pid_t pid, int sig)
```

给pid的进程发送信号。发送进程必须有权限，发送进程的权限等于发送进程的user id。

- pid > 0 信号发送给ID时pid的进程。
- pid = 0 信号发送给跟本进程group ID相同的进程（并且有权限）。
- pid = -1 如果发送进程是root进程，发送给所有的进程；不然发送给所有该user id的进程。
- pid < 0 信号发送给进程组id 等于 pid的进程。（osx中没有这项功能）
- 返回0成功，-1失败。
- sig = 0， 则不会发送信号，仅检测进程是否存在，存在返回0，不然返回-1。

```
int raise(int sig)
```

向当前的进程发送信号。

```
unsigned alarm(unsigned seconds)
```

定时发送SIGALRM，时间单位秒。

- 返回值是之前设置的alarm的遗留值。
- 后面的alarm设置会覆盖前面的alarm设置。
- alarm(0)表示终止定时器。
- 如果不捕捉，默认动作终止进程。

```
int pause(void)
```

挂起进程，直到执行信号处理程序并从中返回，默认返回-1， errno为EINTR。



#### 使用alarm实现sleep

早期的sleep函数使用alarm + pause实现，设置alarm后，pause，alarm信号会使pause返回。
有问题：

- 会打乱正常的用户设置的alarm处理
- 存在竞态，如果系统繁忙，可能alarm在pause前返回。

前者保存之前配置即可；后者可用setjmp或者sigprocmask和sigsuspend解决。
在alarm信号处理函数中jmp到io之前即可。

#### 使用alarm中断过长的阻塞操作

如果一个阻塞io，没有提供timeout，可以使用alarm函数中断。

问题：

- 存在竞态，可能alarm这read操作前返回
- read在系统中可能是自动重启的

同样可以用setlongjmp解决竞态的问题。

## 6 信号集，信号集的操作

为了能使进程操作自己的信号处理方式，定义了一个信号的集合类型 `sigset_t`。

#### 信号集操作

```
#include <signal.h>

//初始化信号集，清空所有信号
int sigemptyset(sigset_t *set);
//初始化信号集，使其包含所有信号
int sigfillset(sigset_t *set);
//增加信号signo到信号集
int sigaddset(sigset_t *set, int signo);
//从信号集删除信号signo
int sigdelset(sigset_t *set, int signo);

//检测信号集是否包含信号signo，是返回1，否返回0
int sigismember(const sigset_t *set, int signo);
```


#### 屏蔽信号 sigprocmask

进程的信号屏蔽字可以设置当前进程阻塞不能递送的信号集。调用sigprocmask函数可以检测设置进程的信号屏蔽字。

```
int sigprocmask(int how, const sigset_t *set, sigset_t *oset);
```

- oset 如果是非空指针则之前的信号屏蔽字通过它返回。
- 如果 set 是非空指针，参数how指示如何修改当前的信号屏蔽字。set是空指针则不做修改。
- 系统默认不屏蔽 SIGKILL SIGSTOP

1. SIG_BLOCK 		将set信号集加入屏蔽字
2. SIG_UNBLOCK 		将set信号集从当前屏蔽字删除
3. SIG_SETMASK 		替换信号屏蔽字为set


#### 查询当前pending的信号

当前进程的信号如果未被处理，则存在两种状态：还没有被系统处理，将要被处理（未决）；被信号屏蔽字阻塞。
通过sigpending函数可以获得这些信号。


```
int sigpending(sigset_t *set);
```

#### sigaction 函数



