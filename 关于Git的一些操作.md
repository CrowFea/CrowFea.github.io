---
title: 关于Git的一些操作
date: 2020-06-02 19:28:12
categories:
    - iOS
tags: 
    - git
mathjax: true
---
虽说之前用过git，但是最近真正开始在团队中干活的时候发现还是基本不会的状态。这篇里面记录一下经常会用到的一些命令。

### 正常的流程
首先从要提交的分支上拉取最新的代码，尽可能的避免会出现的冲突。接下来修改代码，提交到本地分支，推到远端后提出merge request，在commit里@code viewer。如果一切ok的话等到review结束就合并到主分支中。

当然这中间会碰到一些问题，可以参考以下的情况。
<!--more-->
### 暂存更改
比如一个东西改到一半，需要先做另一个高优的事情；或者因为修改过本地的文件无法检出分支，可以先通过`git stash`将目前的更改保存，返回到上一个干净的状态。然后检出，再通过`git stash pop`将更改pop出来。

通过`git stash list`可以查看所有的stash，具体想要pop某一条记录可以通过`git stash pop stash@{0}`这样的语句。注意stash是一个栈的结构。

### 撤销commit
可能commit错了，使用`git reset`可以撤销掉已经提交的commit。`git reset`的本质是让代码返回到上一次的commit。
`git reset --soft`	只会撤销commit，这个文件还存在并且已被追踪，所有的更改不会撤销
`git reset —mixed`这是默认的模式，这个文件还存在，但没有被追踪，需要重新add
`git reset —hard`这个会直接返回到上一次commit，更改的文件也会没有
后面还可以再加上记录号，这样就指定回到了某一个commit的场景。

### 保证个人分支和dev同步
先拉取最新的dev，然后切到自己的本地分支，使用`git rebase dev`。到这一步就在本地全部完成了同步，远端还没有，最后再push上去就可以。

### dev和本地dev冲突
为了解决xcode无法断点等问题，我修改了项目文件，后与新的dev冲突。强制让远端dev覆盖
```
git fetch --all
git reset --hard origin/dev
git pull
```
或者如果可以直接`git stash`所有的更改。`git stash save filename`可以存储特定的文件。