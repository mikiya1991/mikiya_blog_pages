---
layout: markdown	
title: "ubus api 使用指南"
comments: true
tags: ubus uloop openwrt 进程间通讯 线程同步
---

ubus是openwrt的进程间通讯(IPC)的库。libubus其是libubox的IPC通信扩展。

libubox是一个C的事件处理库（uloop），可以支持文件、socket的event，支持timeout定时器，支持runqueue运行队列，支持ustream等魔幻操作。

同类型的C事件库还包括libev、libuv、libevent（libev、libuv还有lua扩展，可在lua中使用）。libubus功能相对比较弱，libuv相对是最强的C事件库。


#### 1.1 ubus api 分为client端和server端

对应的例子可参照 ubus sourcecode目录的example/client.c example/server.c

api参考头文件 libubus.h

#### 1.2 ubus blobmsg的pack与parse

blobdata 是二进制的格式化的通用数据，有点像google的protobuf，可以认为是二进制的json数据。之所以要blobdata可能是因为二进制解析速度块数据量小吧。

对应的例子请参照 libubox source code 目录的example中的 example/blobmsg-example.c

#### 1.3 ubus_send_reply 和 status code

ubus的server handle的reply包含两个部分，一个是返回的blobdata数据，这部分需要客户端去blobmsg_parse; 一个是返回的状态, return的code 0,UBUS_INVALID_ARGMENT etc。

一般来说，若是不需要返回数据，但要返回状态，直接在server的handle中return 0,UBUS_STATUS_INVALID_ARGUMENT, UBUS_STATUS_UNKOWN_ERROR等等ubus定义的status code; 对端的client code 可以直接通过ubus invoke的返回值拿到，比较方便。

若将错误码用 ubus_send_reply返回为blobdata，client端还需解析处理起来比较麻烦；所以无特殊需要返回数据，直接return ubus的错误码较好。

## 2 libubox/ubus是线程不安全的

多线程使用不加锁是会段错误的。因为内部的事件处理用了ubus context中的同一块msgbuf。多线程使用会同时使用该buffer，导致段错误。


## 3 怎样安全的在多线程中使用ubus api

首先ubus的使用可认为包括两部分api。

#### server端的api：


ubus的handler中的所有代码 + ubus_send_rly。该类型代码跑在uloop的文件fd事件的callback函数中，由uloop单线程处理。

不可以多线程，不支持多线程操作。因为server端的callback是uloop中处理，根本上是没有办法加锁的，所以server端的handle代码，天生不能多线程。


但是可以使用 deferd_request在另外的线程中处理，最后用 uloop_timeout 让handle的线程回复。这种魔幻的操作。


#### client 端api：


client端 api 包括ubus_invoke, ubus notify（没错它相当于client端的api）。

client端api是没有跑在uloop中的（没有ubus_add_uloop就不在uloop中），所以不是uloop处理它的callback，因此我们是可以加锁的。

所以client端api可以通过加锁实现多线程安全的使用。


#### server端api和client端api可不可以一起使用？


**答案是可以**

使用的时候，server端的ubus_context 必须与client的ubus_context区分。 就是server端的context只能用在handle中的ubus_send_rly，并且只能在handle的线程。


如果需要在handle函数中invoke其他的ubus，就需要另建一个server用的client的ubus_context用来invoke别的ubus object。

然后这个client的context可以多线程使用，但是必须加锁。



> 注意：某些调用非常频繁的ubus，建议单独使用client的context。因为实际的使用时候有现象表明，其他的ubus的invoke可能会被某个占用很高的ubus invoke卡住（加锁导致的）


### 4 将复杂的ubus任务交给其他线程来做


因为uloop中消耗太多时间会导致uloop被卡住，uloop被卡住就不会响应其他人的请求了，程序的使用感受上就会很差。

解决方案为：

1. 在uloop handle中使用ubus_defered_request，延期答复
2. 在一个处理线程中完成对ubus request数据的处理
3. 然后该线程使用uloop_timeout_set api 设置一个timeout，定时0；让uloop处理该timeout
4. 在timeout中调用 ubus_complete_defered_request api，发出request的reply

> 注意：  
之所以一定要在uloop的线程回复ubus request，是因为一定要用uloop handle的ubus context回复;而这个ubus context线程不安全(上文所说的不能加锁)，只能在该线程处理


### 5 其他线程中设置uloop_time可能永远无法触发


然而还有一个问题，在另外的线程中使用uloop_timeout_set 是会有问题， 可能永远不会运行。

因为可能uloop已经在epoll中卡死了。

>  1. uloop 在epoll开始之前会查看所有的timeout，计算最近的timeout的时间间隔，然后epoll的这个timeout的时间。  
2. 这样存在一个问题，如果uloop已经开始epoll了；而另一线程增加了一个timeout，但是uloop是不知道的；它无法从epoll中退出。  
3. 但是uloop线程中callback中增加timeout是不会有问题的，因为这些个timeout_set总会在epoll之前。

**解决方案是使用一个专门用来唤醒uloop的文件符，加入到uloop的pollfd中**

这样在另外的线程中设置了timeout后，往唤醒fd中写一些数据，强行唤醒epoll。
