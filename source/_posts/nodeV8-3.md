---
title: node/V8小笔记-3
abbrlink: 44ee3b32
date: 2018-01-08 14:20:01
tags:
---

&emsp;&emsp;node/V8学习笔记系列3

<!-- more -->


### cluster
  node的cluster模块封装了创建子进程、进程间通信、服务负载均衡。有两类进程，master和worker，master负责启动worker，worker负责干活。
#### 竞争模型
  master进程创建socket，将fd传递到fork出的worker进程，worker进程接收fd再调用listen，accept新的连接。worker进程之间竞争上岗，谁能accept新进来的连接不确定。
#### round-robin (轮询)
  master创建socket，绑定好地址和端口后监听，新的连接进来以后，将客户端的socket fd传递给指定的worker处理。
