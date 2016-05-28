---
layout: post
title:  "hello pyside!"
date:   2016-05-27 15:49:54
categories: pyside
comments: true
---

* content
{:toc}

### 什么是PySide
PySide由Qt的官方团队--Nokia Qt进行维护，集成了Qt和Python的优势。一个PySide程序员只需要使用简单的Python语言就能够发挥Qt的所有功能。PySide拥有LGPL2.1版授权许可，允许进行免费/开源软件和私有商业软件的开发。Matplotlib，PhotoGrabber，Wing IDE，Lucas Chess，Fminer等应用程序均使用PySide开发，这证明了PySide在软件行业的广泛普及和使用。另外，PySide Mobility工程还允许python访问Qt Mobile API，这对你进行移动开发是很有帮助的。现在让我们开始学习吧！。

### PySide与PyQt的差异
[官法说明](https://wiki.qt.io/Differences_Between_PySide_and_PyQt/zh)

### 怎么安装PySide
OS: ubuntu 16.04 X86_64

- 安装QT

[下载地址](http://www.qt.io/download/)

```
dyn@ubuntu:~$ chmod +x qt-unified-linux-x64-2.0.3-online.run
dyn@ubuntu:~$ ./qt-unified-linux-x64-2.0.3-online.run
```

- 安装PySide(比较省事的方法)

```
dyn@ubuntu:~$ sudo add-apt-repository ppa:pyside
dyn@ubuntu:~$ sudo apt-get update
```
使用Python2.X就安装python-pyside

```
dyn@ubuntu:~$ sudo apt-get install python-pyside
```

使用Python3.X就安装python3-pyside

```
dyn@ubuntu:~$ sudo apt-get install python3-pyside
```

- 使用源码安装

本人没有试过,可参照[官方说明](http://wiki.qt.io/Building_PySide_on_Linux)

- 测试是否安装成功

```
dyn@ubuntu:~$ python3
Python 3.5.1+ (default, Mar 30 2016, 22:46:26) 
[GCC 5.3.1 20160330] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import PySide
>>> 
>>> PySide.__version__
'1.2.2'
>>> 
```


