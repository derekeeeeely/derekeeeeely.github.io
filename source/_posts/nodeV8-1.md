---
title: node/V8小笔记-1
abbrlink: '33612064'
date: 2018-01-06 15:51:30
tags:
---

&emsp;&emsp;Node/V8学习笔记系列1，关于Isolate（V8实例）和GC（垃圾回收机制）

<!-- more -->

## JavaScript引擎
  ![](http://opo02jcsr.bkt.clouddn.com/7b6d799a60dcdda8f8321e54d4cd65db.png)

  js引擎：源代码->抽象语法树AST->字节码->JIT->本地代码
  V8：AST->JIT->本地代码

## V8
### Isolate
&emsp;&emsp;V8引擎实例，是一个独立的虚拟机，对应一个或多个线程，但同时只允许一个线程进入。Isolate之间完全隔离，不共享任何资源。

  ![](http://opo02jcsr.bkt.clouddn.com/c5e9cafc6bd2eb777027f92b22eed647.png)

  - Handle
  &emsp;&emsp;V8对所有js值和对象的内存分配都是在Heap内，Heap由V8独立维护，失去引用的对象会被GC回收。Handle是对Heap中对象的引用，GC需要管理Handle，对象的Handle引用变化时GC可以回收该对象或者移动该对象的分区。
  &emsp;&emsp;Handle分为Local和Persistance两种。前者为局部对象，后者类似全局对象，需要类似于C++的new和delete的操作Persistent::New, Persistent::Dispose进行内存管理。
  &emsp;&emsp;Persistent::MakeWeak可以弱化引用，触发GC对被引用对象的回收。

  - Scope
  &emsp;&emsp;作用域，是Handle（句柄）的容器，一个作用域内可以有很多句柄。
  &emsp;&emsp;HandleScope用于管理Handle，Context::Scope用于管理Context对象

  - Context
  &emsp;&emsp;上下文环境，也可以理解为运行环境。
  &emsp;&emsp;例如：在A函数内有一个Context，调用B函数，又有一个Context，退出B回到A时又恢复了A的Context。

### GC
&emsp;&emsp;基本问题：识别需要回收的内存。
&emsp;&emsp;根对象或者被另一个活跃对象引用的对象是活跃的。

  - V8的堆构成
  ![](http://opo02jcsr.bkt.clouddn.com/dce361c2c729b461e24b5582c56c3d8f.png)

  - 识别指针
  &emsp;&emsp;通过在指针的末位标记这个字是指针还是对象

  - 对象晋升
  &emsp;&emsp;对象在经历了多次新生代的清理后还幸存的时候，会被移动到老生代，即被晋升

  - 跨区指向
  &emsp;&emsp;新对象诞生时并没有指向他的指针，比如在老生区对象写操作的时候，产生了一个队新对象的引用，此时在写缓冲区被记录下这样的跨区指向。

  - GC三步曲
    - 枚举根节点引用
    - 发现并标记活跃对象
    - 垃圾内存清理
    分代回收分两种：Scavenge和Mark-Sweep
      - Scavenge  分配指针达到新生代末尾时清理
      - Mark-Sweep  活跃超过两个小周期的对象，需要将其移动到老生区
