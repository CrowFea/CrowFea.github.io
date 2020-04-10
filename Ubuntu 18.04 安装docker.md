---
title: Ubuntu 18.04 安装docker
date: 2020-04-08 10:26:12
categories:
    - 安装配置
tags: 
    - docker
mathjax: true
---

环境：虚拟机Ubuntu18.04,[【Ubuntu18.04】安装Docker教程_运维_iefenghao的博客-CSDN博客](https://blog.csdn.net/iefenghao/article/details/90747642)

首先更新源
```
sudo apt update
```
安装依赖
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
添加Dokcer官方密钥到系统中
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
添加Docker源
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```
更新一下源
```
sudo apt update
```
查看可以安装的docker版本
```
apt-cache policy docker-ce
```
开始安装docker
```
sudo apt install docker-ce
```
测试
```
docker --version
```
```
sudo docker run hello-world
```

以上安装Docker成功
