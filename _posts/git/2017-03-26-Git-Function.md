---
layout: post
title:  "Git基本功能介绍"
date:   2017-03-26 21:35:54
categories: git
comments: true
share: true
---

* Table of Contents
{:toc}

## clone
克隆，顾名思义就是完全复制出一个一模一样的仓库来，把远端的仓库信息拿到本地。

## fetch
获取，获取什么呢？其实就是和之前克隆的仓库或者是新增的远端仓库进行同步，如果远端仓库有新的提交，则更新本地记录的远端仓库信息，但并不影响工作区内的任何文件。

## merge
合并，就是让两个不同的分支进行历史及代码的合并，把另一个分支合并到当前分支，会改变工作区的文件。

## pull
pull = fetch + merge，先获取远端分支信息，再将其合并到当前分支，当然也就是改变工作的文件了，这个操作也是我们经常更新代码时使用，但它与SVN的update还是不太一样的，也是经常被误解的一个操作，理解pull的工作原理才能更好的使用它。

## checkout
检出，就是把文件从版本库中提取出来，这个功能还是很强大的，可以做很多事情，如：切换分支、创建并切换到新的分支、恢复工作区有变化的文件等。

## add
添加，这个功能就是将有变化（修改、新增、删除）的文件添加到暂存区，这也是git和svn最大的区别——多了一个暂存区的概念，它是一个中间状态，只有被提交到的暂存区的变更才能最终被提交到本地的版本库里。

## commit
提交，上面已经提到了，会将暂存区中的内存提交到本地的版本库中；图形界面TortoiseGit里的commit实际上是add + commit，省略了add的步骤，如果用命令行的就可以更好的理解暂存区的概念了。commit时会通过SHA-1的算法算出一个40位的16进制数字来标识一次提交，svn的话只是一个递增的序号，前段时间在网上看到，linus好像是想改用其它算法了，不知道真假。

## push
push和pull是类似的，都是与远端库进行同步，只是方向不同而已。push(推)自然是将本地提交的commit推送到远端了。

## clean
清理，svn中的clean是将清理版本库的状态，当你做某个操作时意外中止了，再做其它操作时就可能会让你clean一下；而git的功能却完全不同，它是将本地没用的文件清理掉，没用的文件主要是不受版本库控制的文件，这一点也是经常被弄混。

## revert
恢复，这个功能svn和git也是不同的，svn和TortoiseGit的revert是将本地未提交的变更恢复成与版本库的一致，而git是使用checkout与实现的这一功能。那么，git revert是做什么的呢？其实它也是恢复，只不过是恢复之前的某一次提交，即将之前的某一次的修改再给改回去，并生成一个新的提交。

## blame
这个功能就和svn是一样的了，查看文件每一行代码对应的最后一次commit

## cherry-pick
将commit从一个分支提交到另一个分支，会重新生成一个新的SHA值，但是log、提交日期、提交人等信息还是一样的。cherry-pick多个commit的时候，要注意顺序，和之前commit的顺序要一致，否则可能会出现冲突。

## rebase
衍合，可以把在一个分支里提交的改变移到另一个分支里重放一遍。它的原理是回到两个分支最近的共同祖先，根据当前分支（也就是要进行衍合的分支 experiment）后续的历次提交对象，生成一系列文件补丁，然后以基底分支（也就是主干分支 master）最后一个提交对象为新的出发点，逐个应用之前准备好的补丁文件，最后会生成一个新的合并提交对象，从而改写 experiment 的提交历史，使它成为 master 分支的直接下游 。

## reset
在提交层面上，reset将一个分支的末端指向另一个提交。这可以将当前分支指向之前的提交，用来移除当前分支的一些提交。

## stash
隐藏，当你正在改一个bug改到一半的时候，又有一个紧急bug修改，这时怎么办呢？之前改的不要了？不划算啊！这时就用到stash了，将当前修改到的文件先隐藏起来，然后再切换到别的分支修改紧急bug，修改完之后再把stash的内容pop出来，就可以继续修改了。