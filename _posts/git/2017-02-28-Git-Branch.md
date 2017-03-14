---
layout: post
title:  "Linux终端显示Git分支名"
date:   2017-02-28 22:49:54
categories: git
comments: true
share: true
---

* content
{:toc}

### 1.进入你的home目录
```
cd ~
```

### 2.编辑.bashrc文件
```
vi .bashrc
```

### 3.将下面的代码加入到文件的最后
```
function git_branch {
  branch="`git branch 2>/dev/null | grep "^\*" | sed -e "s/^\*\ //"`"
  if [ "${branch}" != "" ];then
      if [ "${branch}" = "(no branch)" ];then
          branch="(`git rev-parse --short HEAD`...)"
      fi
      echo " ($branch)"
  fi
}

export PS1='\u@\h \[\033[01;36m\]\W\[\033[01;32m\]$(git_branch)\[\033[00m\] \$ '
```

### 4.保存退出后，执行加载命令
```
source ./.bashrc
```
