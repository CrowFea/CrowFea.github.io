---
title: Xcode抽疯集锦
date: 2020-07-01 19:28:12
categories:
    - iOS
tags: 
    - Xcode
mathjax: true
---

### 通用解法
- clean
- 重启

### 打不了断点
11.5版本出现。打断点时关掉变量窗口，读变量会挂。
[加这段代码](https://developer.apple.com/forums/thread/134059)

### pod install成功但是找不到变量
把本机的缓存全删干净。
- project clean
- 删掉`/Users/admin/Library/Developer/Xcode/DerivedData`下的全部文件
- pod cache clean —all
- 项目目录下 make clean
到这基本就能解决了， ~~实在不行了重启或者重装~~

### 没有联想
开发一般在pod里面，新增了文件先`pod install`，要不然关联不到