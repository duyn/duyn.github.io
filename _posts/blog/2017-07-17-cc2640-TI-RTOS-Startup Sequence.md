---
layout: post
title:  "CC2640 TI-RTOS SYS/BIOS启动时序"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
feature:
date:   2017-07-17 09:00:54
---

SYS/BIOS启动顺序在逻辑上划分为两个阶段：main()函数之前被调用的和main()函数之后被调用的。启动函数中的每一种启动顺序都在不同地方提供了控制指针供用户操作。

"main()之前" 启动顺序完全由XDCtools运行时包来管理。XDCtools运行时启动顺序如下：
1. CPU重启之后，立即执行具体目标/设备的CPU初始设定（从c_int00开始）。
2. 在cinit()之前运行“reset functions”表（ xdc.runtime.Reset模块提供这个hook）。Reset.fxns[] 数组中的函数被调用。这些重启函数仅在重启时在运行一个程序之前被平台调用。
3. 运行cinit()对C运行时环境进行初始化。
4. 运行用户提供的第一个函数（xdc.runtime.Startup模块提供此hook）。
5. 运行所有模块初始化函数。
6. 运行用户提供的最后一个函数（xdc.runtime.Startup模块提供此hook）。
7. 运行pinit()。
8. 运行main()。

"main()之后" 启动顺序由SYS/BIOS管理，它通过应用程序main()函数最后的BIOS_start()函数显式调用完成初始化。BIOS_start()调用后，SYS/BIOS启动顺序运行如下：
1. Startup Fuctions：运行用户提供的“startup functions”（见BIOS.startupFxns）。如果系统支持计时器，所有静态创建的计时器在此时使用它们的静态配置进行初始化。如果计时器配置为自动启动，它也在此开始计时。
2. Enable Hardware Interrupts（使能硬件中断）。
3. Enable Software Interrupts（使能软件中断）。如果系统支持软件中断（Swis见BIOS.swiEnabled），那么SYS/BIOS启动顺序在此时使能Swis。
4. Task Startup。如果系统支持任务（见BIOS.taskEnabled），那么任务调试始于此处。如果系统无静态或动态创建的任务，那么运行直接进入idle loop。