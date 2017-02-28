---
layout: post
title:  "PySide使用qt设计的UI"
date:   2016-05-27 15:49:54
categories: pyside
comments: true
---

* content
{:toc}

### 使用Qt Creator设计UI
- 打开Qt

![openqt](http://duyn.github.io/png/Pyside_UseQtUI/openqt.png)

- 点击菜单栏File，新建File

![setoption1](http://duyn.github.io/png/Pyside_UseQtUI/setoption1.png)

![setoption2](http://duyn.github.io/png/Pyside_UseQtUI/setoption2.png)

![setoption3](http://duyn.github.io/png/Pyside_UseQtUI/setoption3.png)

最后点击"Finish"，进入主窗口。

![qtdesign](http://duyn.github.io/png/Pyside_UseQtUI/qtdesign.png)

- 插入文本框

![label](http://duyn.github.io/png/Pyside_UseQtUI/label.png)

设计完成后，保存退出。

### 将UI文件转换为Python文件
```
dyn@ubuntu:~/MyPython/HelloPySide$ pyside-uic helloui.ui -o helloui.py
```

如果提示打不到pyside-uic，则需要安装pyside-tools:

```
dyn@ubuntu:~/MyPython/HelloPySide$ sudo apt-get install pyside-tools
```

查看生成了helloui.py文件

```
dyn@ubuntu:~/MyPython/HelloPySide$ ls
Hello.py  helloui.py  helloui.ui
dyn@ubuntu:~/MyPython/HelloPySide$
```

### 编写主函数
代码如下:

![HelloCode](http://duyn.github.io/png/Pyside_UseQtUI/HelloCode.png)

### 最终效果

```
dyn@ubuntu:~/MyPython/HelloPySide$ python3 Hello.py
```

![HelloCode](http://duyn.github.io/png/Pyside_UseQtUI/HelloPySide.png)

注:项目所有代码可到如下路径查看，欢迎批评指正:
[HelloPySide](https://github.com/duyn/MyCode/tree/master/Python/HelloPySide)
