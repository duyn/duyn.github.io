---
layout: post
title:  "CC2640 TI-RTOS BLE Protocol Stack"
modified:
comments: true
share: true
categories: blog
excerpt:
tags: []
image:
feature:
date:   2017-07-18 11:27:15
---

This section describes the functionality of the Bluetooth low energy protocol stack and provides a list of APIs to interface with the protocol stack. The stack project and its associated files serve to implement the Bluetooth low energy protocol stack task.

# Generic Access Profile (GAP) #
+ **Standby:** 设备的初始状态
+ **Advertiser:** 将之前设置好的数据通告给已连接的设备
+ **Scanner:** 收到通告的消息时，正在扫描的设备会发送一个扫描请求给Advertiser，Advertiser会回应答，这个过程就是“设备发现”。扫描的设备知道Advertiser，并可以发起与它建立连接。
+ **Initiator:** 发起者要指定一个要建立连接的对端设备地址。收到Advertiser发的通告，并且地址也匹配，发起者就会发送一个建立连接的请求。
+ **Slave/Master:** 连接建好之后，Advertiser就会成为Slave, Initiator就是Master。

## Connection Parameters（连接参数） ##
+ **Connection Interval：** 两个设备要在一个指定的频道、指定的时间发送、接收数据，设备在满足一定时间之后就会在一个新的频道里交互数据。两个设备发送、接收数据被称为“Connection Event”，如果两个设备之间没有数据交互，会通过交换链路层数据来维持这个连接，Connection Interval就是两个Connection Event之间的时间，单位是1.25ms，可设置的范围是6 (7.5 ms) —— 3200 (4.0 s)。每间隔一个“Connection Interval”就会发送一个Connection Event。
+ **Slave Latency：** 这个参数可以让Slave设备忽略一定数量的Connection Event，外围设备在不发数据的时候，可以忽略Connection Event，保持休眠，节省电量，外围设备可以选择是否在每个“Connection Event”的基础上唤醒。
+ **Supervision Time-out：** 两个成功的“Connection Event”之间最大时间，超过这个时间就会中断连接、返回非连接状态。参数范围是10ms，配置范围：10 (100 ms) to 3200 (32.0 s)。

*Effective Connection Interval = (Connection Interval) × (1 + [Slave Latency])，大部分的参数都是通过GAPRole Task来设置，只有个别参数是通过直接调用GAP的接口调用的。*

##GAPRole Task##
GAPRole Task是一个独立的任务，它独离于APP，处理大多数GAP层的功能。在初始化期，由APP来使能或配置这个任务。很多BLE Protocol Stack事件直接由GAPRole Task处理，不会传送给APP。APP可以向GAPRole Task注册回调函数，以便可以将事件通知给APP Task，并进行相关处理。

GAP层有以下几个角色：
**Broadcaster：** Advertiser 是非连接的状态
**Observer：** 设备在扫描通告，还没有发起连接
**Peripheral：** Advertiser已连接，在连接中作为Slave
**Central：** 设备扫描后发起连接，在连接中作为Master

#Generic Attribute Profile (GATT)#
正如GAP层处理大部分连接相关的功能，GATT层处理的是两个连接的设备之间数据交互的功能。数据是以存在设备内存中“特性”的形式来传递和保存的。站在GATT的角度看，两个设备连接之后，他们就会扮演以下两个角色中的一个：
**GATT server：** 包含GATT client读或者写的数据的设备
**GATT client：** 正在向GATT server写数据或从GATT server读数据的设备
GATT的角色和GAP的角色是相互独立的。

##GATT Characteristics and Attributes##
属性是实际上设备之间传输的信息，Characteristics就是将数据、属性和配置等信息组合起来使用。一个典型的Characteristics包含以下几个属性：
**Characteristic Value:** data value of the characteristic 
**Characteristic Declaration:** 描述了存储的属性、位置和数据类型
**Client Characteristic Configuration:** GATT server可以配置异步发送消息带或不带ack。
**Characteristic User Description:** 一段描述Characteristic的字符串
这些属性都是存在server的一个属性表中的，每个属性除了value外，还包括Handle（表的索引，用户不可设置）、Type、Permissions。

##GATT Services and Profile##
A GATT service 是 characteristics 的集合，多个 services 组成了 a profile. 多个 profiles 只能实现 one service。

##GAP GATT Service(GGS)##
包含以下Characteristics
– Device name
– Appearance
– Peripheral preferred connection parameters

GGS 对于BLE设备来说，可以实现central 或 peripheral角色，多角色的设备要实现这些角色中的一个也需要GGS。GGS在设备发现和连接发起的过程中起来协助的作用。

##Generic Attribute Profile Service##

##GATT Client Abstraction##
GATT层也是抽象的，具体的抽象取决于GATT的角色是Client还是Server，GATT层是ATT层的抽象。

##GATT Server Abstraction##
作为GATT Server，大部分GATT的功能都是通过GATT profiles来处理的，而GATT profiles要使用GattServApp（配置和管理属性表的模块）

GATTServApp存储和管理APP相关的属性表，Profiles通过这个模块添加Characteristic到属性表中,BLE Stack用这个模块响应来自GATT Client的发现请求。
在上电或者重启的时候，APP会通过GATTServApp构建属性表，每个服务都包含一系统的属性(UUIDs, values, permissions, and read and write call-backs)。

##Profile Architecture##
+ **Attribute Table Definition**
GATT属性表要定义成固定大小，然后传递给GATT。
+ **Add Service Function**
+ **Register Application Callback Function**

##GATT Security##
GATT Server可以为每个Characteristic定义单独的权限。
### Authentication ###
需要认证的Characteristics只能被通过配对认证的Client访问，这个校验只需要在协议栈内执行，不需要APP处理。

###Authorization###
授权是可由APP自己定义的安全层，协议栈会将读、写请求发送给应用层，需要注册一个authorization回调函数。这个回调函数会协议栈的上下文执行，因此这个函数不能执行太复杂的操作。如果授权通过，可以访问则返回SUCCESS，如果还没有获得适当的授权则返回ATT_ERR_INSUFFICIENT_AUTHOR。