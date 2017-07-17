---
layout: post
title:  "CC2640 TI-RTOS 支持的模块"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
feature:
date:   2017-07-17 16:55:54
---

This chapter describes modules that provide APIs for some basic support features and configuration parameters that manage overall application behavior.

## BIOS ##
BIOS模块负责设置SYS/BIOS所使用的全局参数，并负责执行SYS/BIOS启动顺序。

BIOS模块的包路径是ti.sysbios.BIOS，所以SYS/BIOS应用将使用以下#include声明：
`#include <ti/sysbios/BIOS.h>`

配置文件（*.cfg）应声明如下：  
`var BIOS = xdc.useModule('ti.sysbios.BIOS');`

以下是SYS/BIOS应用常用的BIOS模块APIs：
+ BIOS_start() — 应用程序的main()函数在把控制权交给SYS/BIOS线程调度前必须调用此函数作为将要执行的最终语句。在所有应用程序所需要的初始化都被执行之后，应调用此函数。此函数执行所有剩余的SYS/BIOS初始化并将控制权转交给准备运行的最高优先级Task。如果Tasks不可用，控制权直接转交给Idle Loop。BIOS_start()函数不会返回。
+ BIOS_getCpuFreq() — 此函数返回以Hz为单位的CPU频率。
+ BIOS_setCpuFreq() — 此函数设置以Hz为单位的CPU频率。
+ BIOS_exit() — 当你希望SYS/BIOS应用程序结束时，调用此函数。BIOS_exit()不返回调用BIOS_start()的那个函数。相反，BIOS_exit()调用System_exit()并中止。所有被System_atexit()或ANSI C标准库atexit()限制的函数都会执行。SYS/BIOS应用程序的BIOS_exit()内会调用System_exit()，因为在调用System_exit()之前BIOS_exit()先执行内部清理。如果应用程序需要因一个错误条件而终止，应调用System_abort()或System_abortSpin()。
+ BIOS_getThreadType() — 此函数返回调用此函数的线程类型。可用的线程类型有BIOS_ThreadType_Hwi，BIOS_ThreadType_Swi，BIOS_ThreadType_Task，和BIOS_ThreadType_Main。

BIOS模块定义了BIOS_WAIT_FOREVER和BIOS_NO_WAIT常量，可用于APIs的timeout参数。

BIOS模块提供了几个配置参数控制全局SYS/BIOS行为。例如，BIOS.heapSize和BIOS.heapSection。BIOS.libType配置参数在SYS/BIOS库建立时用于选择版本。

## System ##
系统模块提供基础的低层次“系统”服务，例如字符输出，printf-like输出和退出处理。

系统模块的包路径是xdc.runtime.System，所以SYS/BIOS应用将使用以下#include声明：
`#include <xdc/runtime/System.h>`

配置文件（*.cfg）将使用以下声明：
`var System = xdc.useModule('xdc.runtime.System');`

以下为SYS/BIOS应用常用的系统模块APIs：
+ System_printf() — 有几个函数提供了类似printf的功能。这些函数包括System_putch()，System_sprintf()，System_vprintf()，System_aprintf() (IArg类型的参数)和system_vsnprintf()（打印指定数量的字符。使用可变参数列表传递参数到字符缓冲）。
我们强力推荐SYS/BIOS应用调用System_printf()和它的相关函数代替标准printf()函数。系统模块提供了熟悉的但拥有更少选项的printf-like函数。内在占用比传统printf()更少。System_printf()允许用户指定要处理的字符输出使用一个系统提供者(System provider)。
相比日志模块打印函数更推荐使用系统模块打印函数，因为系统模块函数更易于使用在各种情况下。日志模块常用于主机仪表工具，它有更多的选项来禁用/可用日志。

+ System_abort() — 当应用程序需要非正常中止，可调用此函数并返回一个错误信息。此函数允许你返回一个描述所发生错误的字符串。当此函数被调用，系统gate将进入，SupportProxy的abort函数被调用，然后调用System.abortFxn。没有绑定System_atexit()或ANSI C标准库函数atexit()的退出函数被调用。

+ System_flush() — 此函数发送所有的缓冲输出字符到输出设备中。它还发出一个断点到IDE。因为这个调用会使目标停止，调用它会影响实时执行的细节。输出字符的目的地由SupportProxy模块决定。

+ System_exit() — SYS/BIOS应用程序推荐使用BIOS_exit()而不是System_exit()函数。BIOS_exit()函数在调用System_exit()前执行内部清理。System_exit()可以在Task内调用，但不能在Swi或Hwi内调用。

+ System_atexit() — 在调用System_exit()时调用此函数为内部执行的函数栈添加一个退出句柄。System.maxAtexitHandlers配置属性控制多少退出句柄可以入栈，默认值为8。

系统模块使用一个支持代理模块实现低层次服务需求。所使用的实现由System.SupportProxy配置参数指定，默认使用SysMin模块。其它选项是SysCallback模块。

系统模块是带gate的。其它模块可使用通过Gate_enterSystem()和Gate_leaveSystem()来使用它们的gate。

**Exit函数**
默认情况下，当应用程序无错误退出，系统模块调用它的标准退出函数，也就是依次调用ANSI C标准exit()函数。如果希望应用程序无限期延长，以调试应用程序状态，在配置中添加以下声明：
System.exitFxn = System.exitSpin;
配置此“spin”函数也减少了应用程序的代码尺寸。System_exitSpin() API将不会被应用程序直接调用。

你也可以在运行时使用C代码System_atexit()函数指定多个退出句柄。一个自定义退出句柄的原型是：
typedef Void (*System_AtexitHandler)(Int)

**Abort函数**
默认情况下，当应用程序因为一个错误条件中止，系统模块调用它的标准abort函数，也就是依次调用ANSI C标准abort()函数。如果希望应用程序无限期延长，以调试应用程序状态，在配置中添加以下声明：
System.abortFxn = System.abortSpin;
配置此“spin”函数也减少了应用程序的代码尺寸。System_abortSpin() API将不会被应用程序直接调用。一个自定义中止句柄原型是：
typedef Void (*System_AbortFxn)();

## SysMin ##
SysMin模块是大多数SYS/BIOS例程和模板中使用得最多的SupportrProxy模块。此模块提供了系统SupportProxy所需要函数实现。其中包含了呈现缓冲字符，输出单个字符和执行退出和中止动作。

大多数应用程序推荐使用SysMin模块，因为它将字符放入环形缓冲，这样RTOS Object Viewer（ROV）工具可以知道如何寻找和呈现。

此模块在目标中维护了一个内部环形缓冲以存储“输出”字符。当缓冲满后，数据重写。当调用函数以呈现缓冲，内部环形缓冲中的字符将使用配置outputFxn“输出”。除非你提供了一个自定义的outputFxn。对于TI目标，TI C运行时支持库使用HOSTwrite()函数输出字符缓冲。对于非TI目标，则使用ANSI C标准库函数fwrite()。

应用程序不应直接调用SysMin函数，但SysMin模块提供配置参数以控制缓存站点用于存储内部输出及包含此缓存的内存片段。配置文件（*.cfg）可通过以下方式使用此模块：
`var BIOS = xdc.useModule('xdc.runtime.SysMin');`

## SysCallback ##
SysCallback是SysMin代理模块的替代模块。它需要你提供自定义函数来处理abort、exit、flush、putch和ready动作。如果需要在应用程序中定制输出行为，则使用此模块。

## Program ##
Program模块作为配置命名空间的“根”。它在配置中使用，但不提供任何C APIs。
配置文件无需useModule()声明来启用此模块。
Program模块提供配置参数来控制运行时内存尺寸。包括Program.stack，控制应用栈尺寸，它由单个Task分别使用。

Program模块还提供配置参数控制自动生成链接命令文件的内容。这包括Program.sectMap[]数组和Program.sectionsExclude参数。

参考API参考帮助（CDOC）的xdc.cfg.Program模块获取有关APIs和配置参数的详细内容。

## Startup ##
Startup模块管理最早期的C的main()函数调用前的启动初始化。这些初始化通常有以下内容组成：设置硬件，指定控制看门狗定时器的寄存器，访问内存，缓存设置，时钟速度等等。

除了允许用户添加自定义启动函数的配置参数外，此模块还提供允许模块自动运行、添加初始化函数到启动顺序的服务。

配置文件（*.cfg）使用如下声明启用Startup模块：
`var BIOS = xdc.useModule('xdc.runtime.Startup');`

## Reset ##
Reset模块使得你通过配置Reset.fxns[]数组属性来定义一个初始reset函数。Reset函数早在应用程序启动时及指定平台硬件初始化时就调用了。

注意只有特定系列的目标在运行程序前执行设备的reset。因此，reset函数并不支持所有平台。不要在这个函数中放置您打算携带的代码。反而应使用Startup模块来定义跨平台应用启动函数。

配置文件（*.cfg）使用如下代码启用Reset模块：
var BIOS = xdc.useModule('xdc.runtime.Reset');
Reset模块允许你配置一个数组的启动函数，让你可为各种外设启动活动提供不同函数。

## Error ##
Error模块提供用于在程序中引发、检查和处理错误的机制。Error模块的包路径是xdc.runtime.Error，所以使用Error APIs的SYS/BIOS应用程序就包含以下#include声明：
`#include <xdc/runtime/Error.h>`

配置文件（*.cfg）应使用如下声明包含Error模块：
`var BIOS = xdc.useModule('xdc.runtime.Error');`

数个SYS/BIOS APIs ，尤其是那是创建对象和分配内存的APIs，都有一个参数为Error_Block类型。
下面为SYS/BIOS应用程序经常用到的Error模块APIs：
+ Error_init() — 将错误块加入它的初始状态。
+ Error_check() — 如果引发了一个错误，则返回true。
+ Error_getData() — 获取一个错误的参数列表。
+ Error_getMsg() — 获取一个错误的“printf”格式字符串。
+ Error_print() — 使用System_printf()打印一个错误。
+ Error_raise() — 引发一个错误。

## Text ##
Text模块高效管理携带公共子串的字符串集合。集合拥有高度公共性，所使用的存储空间远远少于使用普通表格存储独立字符串。使用压缩表示的文本字符串的所有可用空间被限制在64K字符内。

为了进一步节省空间，“压缩”表示甚至无需被载入目标内存。默认情况下，文本字符串是载入目标内存的。你可以在运行时使用Text_isLoaded布尔常量来控制格式化串是否传递给System_printf()函数。

Text模块的包路径是xdc.runtime.Text，所以使用Text APIs和常量的SYS/BIOS应用程序应包含如下#include声明：
`#include <xdc/runtime/Text.h>`

配置文件（*.cfg）应使用如下声明：
`var BIOS = xdc.useModule('xdc.runtime.Text');`