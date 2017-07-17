---
layout: post
title:  "CC2640 TI-RTOS 线程同步机制"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
feature:
date:   2017-07-17 09:43:54
---


## Software Architecture ##

+ RTOS
+ Application Image
+ Protocol Stack Image


## RTOS ##

实时、抢占式、多线程的系统，有如下4种不同优先级的线程（优先级由高到低）：
+ Hardware interrupt service routines (ISRs\Hwi)
+ Software interrupt routines（Swi）
+ Tasks
+ Background idle functions(Idle)

## 线程同步机制 ##
+ 信号量（Semaphores）
分为计数信号量（counting semaphores）和二元信号量（Binary semaphores），二元信号量只有可用和不可用两种状态，计数信号量可记录资源数量。
```
Semaphore_Handle Semaphore_create(Int count, Semaphore_Params *attrs Error_Block *eb );
Void Semaphore_delete(Semaphore_Handle *sem);
Bool Semaphore_pend(Semaphore_Handle sem, UInt timeout); // 请求资源
Void Semaphore_post(Semaphore_Handle sem);	// 释放或发送资源
```
+ 事件（Event）
由一个二进制位来标识一个事件，一个任务可以等多个事件，事件的行为更像是一个二元信息量。Event_pend()可以带AndMask(多个事件同时满足)或OrMask（满足其中一个事件即可），只有任务可以调用Event_pend()，Hwis、Swis、Tasks可以调用Event_post()。
Note: Only a single Task can pend on an Event object at a time.
```
UInt Event_pend(Event_Handle event, UInt andMask, UInt orMask, UInt timeout);
Void Event_post(Event_Handle event, UInt eventIds);
```
+ 门（Gates）
用来保护并发访问的代码关键数据区域（全局变量），防止线程被抢占后，同时修改同一资源，Gate支持嵌套。
*基于抢占的Gate实现：*
**GateHwi**
GateHwi是启用和禁用中断的锁定机构。这种门保证了CPU的独占访问。此门可用于关键区域由Task、Swi或Hwi线程共享时。从进入到离开所需时间应尽可能短于最小Hwi延迟。
**GateSwi**
GateSwi是启用和禁用软件中断的锁定机构。此门可用于关键区域由Task、Swi线程共享时。此门不能用于Hwi线程。从进入到离开所需时间应尽可能短于最小Swi延迟。
**GateTask**
GateSwi是启用和禁用task的锁定机构。此门在关键区域由Task线程共享时使用。此门不能被Hwi和Swi线程使用。从进入到离开所需时间应尽可能短于最小Task延迟。
*基于信号量的Gate实现：*
**GateMutex**
GateMutex使用了一个二元信号量作为锁定机构。每个GateMutex实例都有其自己唯一的信号量。因为此门有可能阻塞，它不能用于Swi或Hwi线程，只能被Task线程使用。
**GateMutexPri**
GateMutexPri是一种互斥Gate（同一时间只能被一个线程操作）。它实现了“优先权继承性”以防止优先反转。优先反转发生于高优先级Task在等待低优先级Task的Gate控制而导致的事实上的“反转”。
+ 信箱（mailboxes）
ti.sysbios.knl.Mailbox模块提供了一系列函数管理mailboxes。Mailboxes可用于在相同处理器的两个task中传递缓冲。一个Mailbox实例可用于多个readers和writers。
Mailbox模块拷贝缓冲到合适尺寸的内部缓冲。这些缓冲的尺寸和个数在Mailbox实例创建时指定。拷贝在通过Mailbox_post()发送缓冲时建立。另外一份拷贝在通过Mailbox_pend()接收缓冲时建立。
+ 队列（Queues）
队列实现为双向链表，所以元素在列表中可以插入和删除，另外，队列没最大尺寸限制。队列有一个“头”，在列表的最前端。Queue_enqueue()将元素添加到列表未尾，Queue_dequeue()移除并返回列表前端元素，这些函数支持队列的FIFO操作。
Queue_head()返回队列的前端元素且不移除它，Queue_next()和Queue_prev()分别返回队列中的下一个和前一个元素。
使用Queue_insert()和Queue_remove()可以在队列中的任何位置插入或移除元素。Queue_insert()在指定元素前插入一个元素，Queue_remove()删除一个指定元素，无论它在队列中的哪个位置。注意，队列不提供任何APIs用于在给定索引入插入或移除元素。
队列通常在系统中通常由多个线程共享，这可能导致不同线程同时更改队列，这将破坏队列。上述讨论的队列APIs不保护这些。然而，队列提供了两个原子APIs，它可以在操作队列前禁用中断。Queue_get()是Queue_dequeue()的原子操作版本，Queue_put()是Queue_enqueue()的原子操作版本。