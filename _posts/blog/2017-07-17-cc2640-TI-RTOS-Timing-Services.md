---
layout: post
title:  "CC2640 TI-RTOS 时间服务"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
feature:
date:   2017-07-17 13:35:54
---


## Clock ##
负责内核用于保持时间轨道的周期性系统tick。所有SYS/BIOS APIs都会用一个Timeout参数来表示根据clock ticks设置的timeout。
时钟模块默认情况下使用 ti.sysbios.hal.Timer模块所创建的计时器来产生系统tick，基于周期性调用Clock_tick()。

时钟模块可被配置为不使用计时器，以下两种静态声明都可以：
```
ti.sysbios.knl.Clock.tickSource = Clock.tickSource_USER
or
ti.sysbios.knl.Clock.tickSource = Clock.tickSource_NULL
```
系统tick的周期由配置参数Clock.tickPeriod进行设置，单位为微秒。

所有时钟函数在Swi上下文中运行。也就是时钟模块自动创建一个Swi并在Swi内运行时钟函数。可以通过配置Clock.swiPriority来改变时钟所使用的Swi的优先级。

你可以使用Clock_create()动态创建时钟实例。时钟实例可以是单次触发或周期触发的。你可以在时钟创建或启动后通过调用Clock_start()来开始一个时钟实例。这由配置参数startFlag来进行控制。Clock_create()仅能在Task或main()函数上下文中被调用。

Clock_create()需要一个函数和一个非0 timeout值作为参数。当timeout到期，函数被调用。timeout值用于计算第一个到期时间。对于单次触发时钟实例来说，timeout值用于计算单个到期时间，period为0。对于定期时钟实例来说，timeout值用于计算第一次到期时间，period值（参数的一部分）用于第一次触发之后。

## Timer ##
提供使用计时器外设的标准接口。它隐藏了计时器外设的所有目标/设备的具体特征。计时器的目标/设备具体属性由 ti.sysbios.family.xxx.Timer模块支持（例如，ti.sysbios.family.c64.Timer）。你可以使用计时器模块来在计时器到期时调用一个tickFxn。

Timer模块是一个面向定时器外设的标准接口，是硬件抽象层（HAL）的一部分。

你可以使用此模块创建定时器（就是标记使用的定时器）并配置为定时器到期时调用一个tickFxn。仅在定时器外设无需任何自定义配置时使用此模块。

计时器可被配置为单周期或多周期模式。period可被指定为定时器次数或微秒。

## Seconds ##
提供了一种手段用于维护当前时间和日期，定义为自1970年（Unix时代）以来的秒数。此模块生成一个自定义的time()函数来调用Seconds_get()，重写了C标准库的time()函数。Seconds模块如果可用的话，将通过一个设备指定的Seconds模块来维护时间。如果设备指定的Seconds模块不可用，ti.sysbios.hal.SecondsClock模块将作为Seconds使用。SecondsClock内部使用时钟模块周期性地增加秒数。

Seconds模块的APIs是：
```
Void Seconds_set(UInt32 seconds);
UInt32 Seconds_get(Void);
```
应用程序应该调用Seconds_set()来初始化Seconds数量。如果需要，Seconds_set()可被再次调用以更新或重设Seconds数量。应用在调用Seconds_get()之前必须至少调用一次Seconds_set()。否则返回结果毫无意义。Seconds_set()函数是非可重入的。

Seconds模块包括一个time()函数，它调用Seconds_get()。它重写了C标准库time()函数。你可使用此time()函数联合C标准头文件（time.h）中的其它函数来以一种可读格式显示当前时间和日期。
Ex:
```C
#include <time.h>
#include <ti/sysbios/hal/Seconds.h>
UInt32 t;
time_t t1;
struct tm *ltm;
char *curTime;
/* set to today’s date in seconds since Jan 1, 1970 */
Seconds_set(1412800000); /* Wed, 08 Oct 2014 20:26:40 GMT */
/* retrieve current time relative to Jan 1, 1970 */
t = Seconds_get();
/*
 * Use overridden time() function to get the current time.
 * Use standard C RTS library functions with return from time().
 * Assumes Seconds_set() has been called as above
 */
t1 = time(NULL);
ltm = localtime(&t1);
curTime = asctime(ltm);
System_printf("Time(GMT): %s\n", curTime);
```

## Timestamp ##
该模块，正如其名字所言，提供时间戳服务。Timestamp模块可用于基准测试代码和添加时间戳日志。调用Timestamp模块函数有利于指定平台的TimestampProvider实施。

Timestamp模块的包路径是xdc.runtime.Timestamp，所以SYS/BIOS应用将包含以下#include声明：
```
#include <xdc/runtime/Timestamp.h>
```

以下Timestamp模块APIs是SYS/BIOS应用中经常使用的：
```
Timestamp_get32() — 返回一个32bit时间戳。
Timestamp_get64() — 如果目标支持64bit解决方案，返回一个64bit时间戳。
Timestamp_getFreq() — 获取timestamp定时器的频率（Hz）。你可以使用此函数将timestamp值转换为实时单位。
```
如果你需要一个平台依赖版本的Timestamp，可以使用TimestampStd模块，它使用的是ANSI C clock()函数。