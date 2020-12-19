# 1.14 scheduler

## 信号概念

信号是一种发给进程的通知，以告知有事件发生。有时信号也被称为软件中断。从中断用户控制流来说，信号和硬件中断是类似的；在大多场景下，信号何时到达进程是无法预测的。

信号可以由内核发给用户进程，可以用户进程发给自己，也可以用户进程发给其它的用户进程。

在进程内部，信号还可以发给某个具体的线程。在 Go 语言中，抢占的信号就是发给指定的线程的。

## 信号是易失的么

信号并不是易失的

Signals appeared in very early UNIX implementations, but have gone through
some significant changes since their inception. In early implementations, signals
could be lost (i.e., not delivered to the target process) in certain circumstances. Furthermore, although facilities were provided to block delivery of signals while critical
code was executed, in some circumstances, blocking was not reliable. These problems
were remedied in 4.2BSD, which provided so-called reliable signals. (One further
BSD innovation was the addition of extra signals to support shell job control, which
we describe in Section 34.7.)
System V also added reliable semantics to signals, but employed a model
incompatible with BSD. These incompatibilities were resolved only with the arrival
of the POSIX.1-1990 standard, which adopted a specification for reliable signals
largely based on the BSD model.

signal mask

pending signal

## 不丢失不代表每次收到的信号都会被触发

信号没有队列 Signals Are Not Queued

The set of pending signals is only a mask; it indicates whether or not a signal has
occurred, but not how many times it has occurred. In other words, if the same signal is generated multiple times while it is blocked, then it is recorded in the set of
pending signals, and later delivered, just once. (One of the differences between
standard and realtime signals is that realtime signals are queued, as discussed in
Section 22.8.

## 信号处理

![](../images/signal.png)

信号处理涉及两个 syscall:

* tigkill

* sigaction

* sigaltstack

## 流程概述

抢占流程主要有两个入口，GC 和 sysmon。

TODO，control flow pic show


### 简单的信号处理函数

简单起见，这里我们使用 C 来做演示:

## gsignal

gsignal 是一个特殊的 goroutine，类似 g0，每一个线程都有一个，创建的 m 的时候，就会创建好这个 gsignal，在 linux 中为 gsignal 分配 32KB 的内存:

newm -> allocm -> mcommoninit -> mpreinit -> malg(32 * 1024)

在线程处理信号时，会短暂地将栈从用户栈切换到 gsignal 的栈，执行 sighandler，执行完成之后，会重新切换回用户的栈继续执行用户逻辑。

## 现场保存和恢复

### GC 抢占流程

suspendG

resumeG

### sysmon 抢占流程

preemptone -> asyncPreempt -> globalrunqput

## 参考资料

1. The Linux Programming Interface
