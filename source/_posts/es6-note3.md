---
title: ES6读书笔记-进阶篇之PG-A
tags:
  - es6
  - javaScript
categories: note
abbrlink: 93cdd49
date: 2017-11-07 13:41:34
---

![](http://opo02jcsr.bkt.clouddn.com/e95028e0591d74b0a2362f5bcf7f8ea6.jpg)
<!-- more -->

> 本文为阅读阮一峰老师的Es6入门一书时做的笔记积累，记录下来的多为个人觉得有必要注意的点，看官老爷们随意就好。

## lesson1 Promise

### 你的名字

- `Promise` 诞于社区，初为异步编程之解决方案，后有ES6将其写入语言标准，终成今人所言之 `Promise` 对象
    - Promise对象特点有二：状态不受外界影响、一旦状态改变后不会再次改变

### 基本用法

- Promise对象为一个构造函数，用于生成Promise实例
    ```js
    // 接受一个函数作为参数，函数参数有两个，为js引擎内部实现的两个函数
    let pr = new Promise(function(resolve, reject) {
      if () {
        resolve() // pending -> resolved
      } else {
        reject() // pending -> rejected
      }
    })
    ```

- Promise实例生成后可以使用then方法指定两种状态下的回调函数
    ```js
    // 第二个函数可选
    pr.then( function(value) {
      // do something
    }, function(error) {
      // throw error
    })
    ```

- so...
    封装一个函数，返回一个Promise对象，在Promise实例创建时传入的函数参数内进行逻辑处理（决定何时改变状态并传递值到回调函数）
    利用Promise实例的then方法接收上一步的传递值处理后续（resolve传递的值可以为Promise对象，reject传递的值多为Error对象）
    - 注意：resolve或reject并不会终结 Promise 的参数函数的执行，因为立即resolve或reject的Promise是在本次时间循环尾部，晚于本轮的同步任务
    - 注意：一般resolve或者reject之后Promise任务完成了，后续应该写在then的回调函数内，所以可以这样`return resolve(value)`

### 我愚蠢的孩子们

#### Promise.prototype.then

- 参数：两个函数，后一个可选

- 返回：新的Promise实例
    可以采用链式写法，上一个回调函数的返回值会作为参数传递到下一个回调函数内
    若上一个回调函数返回的是Promise则需要等状态改变再调用下一个回调函数
    ```js
    getJSON("/post/1.json").then(
      // 返回的是一个Promise，其状态改变后根据状态选择调用下面的then中的哪个回调函数
      post => getJSON(post.commentURL)
    ).then(
      comments => console.log("resolved: ", comments),
      err => console.log("rejected: ", err)
    );
    ```

#### Promise.prototype.catch

- 实际上：是`.then(null, rejection)`的别名
- 链式写法最后一个catch可以捕获前面任何一个Promise对象then方法抛出的error
- 推荐使用catch，而不是两个回调函数作为参数
- 若没有指定错误处理的回调，Promise会吃掉错误，不会退出进程、终止脚本运行
- catch方法里还可以再抛错，再链式catch...

#### Promise.all

- 参数： 具有Iterator接口
- 返回： Promise实例们
    ```js
    var p = Promise.all([p1, p2, p3]);
    ```
    p1, p2, p3都为fullfilled时p才fullfilled，其中一个rejected则p就rejected
    如果其中一个Promise实例定义了catch，错误会被自己的catch捕获然后返回一个新的Promise实例，在执行完catch后该实例也resolved，所以不会被外部p的catch捕获到，若没有，当然会被外面捕获到啦

#### Promise.race

- 概述：同样是多个Promise实例，第一个状态改变的Promise实例会使得p状态跟着改变
- 用处🌰：一定时间内获取不到就rejected
    ```js
    const p = Promise.race([
      fetch('/resource-that-may-take-a-while'),
      new Promise(function (resolve, reject) {
        setTimeout(() => reject(new Error('request timeout')), 5000)
      })
    ]);
    p.then(response => console.log(response));
    p.catch(error => console.log(error));
    ```

#### Promise.resolve/Promise.reject

- 嗯
    ```js
    Promise.resolve('foo')
    // 等价于下面，即生成一个Promise实例，状态为resolved，且将值传给回调函数
    new Promise(resolve => resolve('foo'))
    ```
- 情况
    - 参数为thenable对象，即具有then方法的对象 =>
        将对象封装为Promise对象，立即执行原对象的then方法，然后在根据状态变化去调用Promise.then的回调函数
    - 不是thenable对象或者不是对象
        参见`嗯`
    - 不传参数，返回一个resolved的Promise对象

- 注意：立即resolve的Promise对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时
- 注意：Promise.reject(res)有一点不同，在于参数res直接作为rejected的理由，原封不动地被catch

#### Promise.done/Promise.finally

- 前者表示无论如何都会接收到可能的错误，全局抛出，后者是无论如何都会在最后执行传入的callback
    ```js
    // done
    Promise.prototype.done = function (onFulfilled, onRejected) {
      this.then(onFulfilled, onRejected)
        .catch(function (reason) {
          // 抛出一个全局错误
          setTimeout(() => { throw reason }, 0);
        });
    };
    // finally
    Promise.prototype.finally = function (callback) {
      let P = this.constructor;
      return this.then(
        value  => P.resolve(callback()).then(() => value),
        reason => P.resolve(callback()).then(() => { throw reason })
      );
    };
    ```

#### Promise.try

- 提案：模拟try，是异步就异步，是同步就同步，就提提，再说


## lesson2 Generator

### 初探

#### 概念

- 一个遍历器生成函数，一个状态机
- 调用Generator函数，返回一个遍历器，代表Generator函数的内部指针（此时yield后的表达式不会求值）
- 每次调用next方法会执行下一个yield前的语句并且返回一个{value, done}对象，其中value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值，done属性是一个布尔值，表示是否遍历结束
- 若没有yield了，next执行到函数结束，并将return结果作为value返回，若无return则为undefined。
- 这之后调用next将返回{value: undefined, done: true}

#### yield

- 惰性
    调用next方法时，将yield后的表达式的值作为value返回，只有下次再调用next才会执行这之后的语句，达到了暂停执行的效果，相当于具备了一个惰性求值的功能

- 没有yield时，Generator函数为一个单纯的暂缓执行函数（需要调用next执行）
- yield只能用于Generator函数

#### Iterator接口

- Generator函数为遍历器生成函数，可以赋给对象的Symbol.iterator方法，本身也具有Symbol.iterator属性
    ```js
    function* gen(){
      // some code
    }
    var g = gen();
    g[Symbol.iterator]() === g
    // true
    ```

#### for...of

- Generator函数返回的是遍历器对象，可以直接用for...of访问
- 注意：done为true的会被for...of忽略
- 利用Generator为对象实现Iterator接口
    ```js
    function* objectEntries() {
      let propKeys = Object.keys(this);
      for (let propKey of propKeys) {
        yield [propKey, this[propKey]];
      }
    }
    let jane = { first: 'Jane', last: 'Doe' };
    jane[Symbol.iterator] = objectEntries;
    for (let [key, value] of jane) {
      console.log(`${key}: ${value}`);
    }
    // first: Jane
    // last: Doe
    ```

### 方法

#### `Generator.prototype.next()`

- 参数
    通过传入参数为Generator函数内部注入不同的值来调整函数接下来的行为
    第一次next传递的参数会被忽略（实在想传得包一层）
    ```js
    // 这里利用参数实现了重置
    function* f() {
      for(var i = 0; true; i++) {
        var reset = yield i;
        if(reset) { i = -1; }
      }
    }
    var g = f();
    g.next() // { value: 0, done: false }
    g.next() // { value: 1, done: false }
    // 传递的参数会被赋值给i（yield后的表达式的值(i)），然后执行var reset = i赋值给reset
    g.next(true) // { value: 0, done: false }
    ```

#### `Generator.prototype.throw()`

- Generator函数返回的对象都具有throw方法，用于在函数体外抛出错误，在函数体内可以捕获（只能catch一次）
- 参数可以为Error对象
- 如果函数体内没有部署try...catch代码块，那么throw抛出的错会被外部try...catch代码块捕获，如果外部也没有，则程序报错，中断执行
- throw方法被内部catch以后附带执行一次next
- 函数内部的error可以被外部catch
- 如果Generator执行过程中内部抛错，且没被内部catch，则不会再执行下去了，下次调用next会视为该Generator已运行结束
- Generator函数返回的对象在被next访问完后内部属性`[[GeneratorStatus]]`值变为'closed'了

#### `Generator.prototype.return()`

- `try ... finally`存在时，return会在finally执行完后执行，最后的返回结果是return方法的参数，这之后Generator运行结束，下次访问会得到{value: undefined, done: true}
- `try ... finally`不存在时，直接执行return，后续和上一条一致

#### 实际上

- 以上三种方法都是让Generator恢复执行，并用语句替换yield表达式

### `yield*`表达式

- 在一个Generator内部直接调用另一个Generator是没用的，如果需要在一个Generator内部yield另一个Generator对象的成员，则需要使用`yield*`
    ```js
    function* inner() {
      yield 'a'
      // yield outer() // 返回一个遍历器对象
      yield* outer() // 返回一个遍历器对象的内部值
      yield 'd'
    }
    function* outer() {
      yield 'b'
      yield 'c'
    }
    let s = inner()
    for (let i of s) {
      console.log(i)
    } // a b c d
    ```

- `yield*`后跟一个遍历器对象（所有实现了iterator的数据结构实际上都可以被`yield*`遍历）

- 被代理的Generator函数如果有return，return的值会被for...of忽略，所以next不会返回，但是实际上可以向外部Generetor内部返回一个值，如下：
    ```js
    function *foo() {
      yield 2;
      yield 3;
      return "foo";
    }
    function *bar() {
      yield 1;
      var v = yield *foo();
      console.log( "v: " + v );
      yield 4;
    }
    var it = bar();
    it.next()
    // {value: 1, done: false}
    it.next()
    // {value: 2, done: false}
    it.next()
    // {value: 3, done: false}
    it.next();
    // "v: foo"
    // {value: 4, done: false}
    it.next()
    // {value: undefined, done: true}
    ```

- 举个🌰
    ```js
    // 处理嵌套数组
    function* Tree(tree){
      if(Array.isArray(tree)){
        for(let i=0;i<tree.length;i++) {
          yield* Tree(tree[i])
        }
      } else {
        yield tree
      }
    }
    let ss = [[1,2],[3,4,5],6,[7]]
    for (let i of Tree(ss)) {
      console.log(i)
    } // 1 2 3 4 5 6 7
    // 理解for ...of 实际上是一个while循环
    var it = iterateJobs(jobs);
    var res = it.next();
    while (!res.done){
      var result = res.value;
      // ...
      res = it.next();
    }
    ```

### Extra

#### 作为对象的属性的Generator函数

- 写法很清奇
    ```js
    let obj = {
      * sss() {
        // ...
      }
    }
    let obj = ={
      sss: function* () {
        // ...
      }
    }
    ```

#### Generator函数的this

- Generator函数返回的是遍历器对象，会继承prototype的方法，但是由于返回的不是this，所以会出现：
    ```js
    function* ss () {
      this.a = 1
    }
    let f = ss()
    f.a // undefined
    ```
- 想要在内部的this绑定遍历器对象？js灵活特性尽显...
    ```js
    function * ss() {
      this.a = 1
      yield this.b = 2;
      yield this.c = 3;
    }
    let f = ss.call(ss.prototype) // 由于f.__proto__ === ss.prototype，传入遍历器对象的隐藏原型给this
    f.next()
    f.next()
    f.a // 1
    f.b // 2
    f.c // 3
    ```

#### 应用

- 举个🌰
    ```js
    // 利用暂停状态的特性
    let clock = function* () {
      while(true) {
        console.log('tick')
        yield
        console.log('tock')
        yield
      }
    }
    ```

- 异步操作的同步化表达
    ```js
    // Generator函数
    function* main() {
      var result = yield request("http://some.url");
      var resp = JSON.parse(result);
        console.log(resp.value);
    }
    // ajax请求函数，回调函数中要将response传给next方法
    function request(url) {
      makeAjaxCall(url, function(response){
        it.next(response);
      });
    }
    // 需要第一次执行next方法，返回yield后的表达式，触发异步请求，跳到request函数中执行
    var it = main();
    it.next();
    ```

- 控制流管理
    ```js
    // 同步steps
    let steps = [step1Func, step2Func, step3Func];
    function *iterateSteps(steps){
      for (var i=0; i< steps.length; i++){
        var step = steps[i];
        yield step();
      }
    }
    // 异步后续讨论
    ```

- 部署Iterator接口
- 看做数组结构使用


## lesson3 Generator的异步应用

### 前言

#### 异步

- 一个任务拆分成两个阶段，比如任务是读取一个文件并返回一个结果a。拆分为向操作系统发起请求，然后执行别的操作，再在操作系统返回文件以后去返回一个结果a。不连续的执行，称为异步

#### 回调函数

- 传统的异步实现是通过将第二阶段的操作以函数形式作为参数传递到任务处理方法内，在第一阶段的执行有了结果以后，回调函数才会执行
- node约定，回调函数的第一个参数必须为错误对象，因为第一阶段执行完后，任务得上下文环境结束了。这之后抛出的错误需要传入回调函数才能被捕获处理
- Promise其实只是写法上的区别，依然是以回调函数的方式实现异步

#### Generator函数

- 协程coroutine
    协程A执行->协程A暂停，执行权转交给协程B->一段时间后执行权交还A->A恢复执行
    ```js
    // yield是异步两个阶段的分割线
    function* asyncJob() {
      // ...其他代码
      var f = yield readFile(fileA);
      // ...其他代码
    }
    ```

- Generator函数实现协程
    Generator函数实际上可以作为异步任务的容器，在异步任务需要暂停的地方加上yield注明即可

- Generator函数的错误处理和数据交换
    next方法使得Generator可以接收外部参数，next返回值的value属性是Generator向外的输出值（数据交换实现）
    错误处理上一课有讲述

### Thunk函数

#### 参数的求值策略

- 传名调用和传值调用之争
- 后者更简单，但是可能会有需要大量计算求值却没有用到这个参数的情况，造成性能损失

#### js中的Thunk函数

- 传统的Thunk函数是传名调用的一种实现，即将参数作为一个临时函数的返回值，在需要用到参数的地方对临时函数进行求值
- js中的Thunk函数略有不同
    js中的Thunk函数是将多参数函数替换为单参数函数（这个参数为回调函数）
    ```js
    const Thunk = function(fn) {
      return function (...args) {
        return function (callback) {
          return fn.call(this, ...args, callback);
        }
      };
    };
    ```
    看起来只是换了个样子，好像并没有什么用

#### Thunk函数实现Generator函数自动执行

- Generator函数自动执行
    ```js
    function* gen() {
      yield a // 表达式a
      yield 2
    }
    let g = gen()
    let res = g.next()
    while(!res.done) {
      console.log(res.value)
      res = g.next() // 表达式b
    }
    ```
    > 但是，这不适合异步操作。如果必须保证前一步执行完，才能执行后一步，上面的自动执行就不可行。
    ~~上面这句话是不是可以这样理解，比如表达式a是一个异步操作，回调函数中设置了延时执行，而这个回调函数执行前表达式b可能已经被执行了，进入到下一个循环。所以无法保证执行顺序，所以需要管理回调函数~~
    next方法是同步的，执行时必须立刻返回值，yield后是同步操作当然没问题，是异步操作时就不可以了。处理方式就是返回一个Thunk函数或者Promise对象。此时value值为该函数/对象，done值还是按规矩办事。
    ```js
    var g = gen();
    var r1 = g.next();
    // 重复传入一个回调函数
    r1.value(function (err, data) {
      if (err) throw err;
      var r2 = g.next(data);
      r2.value(function (err, data) {
        if (err) throw err;
        g.next(data);
      });
    });
    ```

- Thunk函数的自动流程管理
    思路：Generator函数中yield 异步Thunk函数，通过yield将控制权转交给Thunk函数，然后在Thunk函数的回调函数中调用Generator的next方法，将控制权交回给Generator。此时，异步操作确保完成，开启下一个任务。
    Generator是一个异步操作的容器，实现自动执行需要一个机制，这个机制的关键是控制权的交替，在异步操作有了结果以后自动交回控制权，而回调函数执行正是这么个时间点。
    ```js
    // Generator函数的执行器
    function run(fn) {
      let gen = fn()
      // 传给Thunk函数的回调函数
      function cb(err, data) {
        // 控制权交给Generator，获取下一个yield表达式（异步任务）
        let result = gen.next(data)
        // 没任务了，返回
        if (result.done) return
        // 控制权交给Thunk函数，传入回调
        result.value(cb)
      }
      cb()
    }
    // Generator函数
    function* g() {
      let f1 = yield readFileThunk('/a')
      let f2 = yield readFileThunk('/b')
      let f3 = yield readFileThunk('/c')
    }
    // Thunk函数readFileThunk
    const Thunk = function(fn) {
      return function (...args) {
        return function (callback) {
          return fn.call(this, ...args, callback);
        }
      };
    };
    var readFileThunk = Thunk(fs.readFile);
    readFileThunk(fileA)(callback);
    // 自动执行
    run(g)
    ```

### co模块

#### 说明

- 不用手写上述的执行器，co模块其实就是将基于Thunk函数和Promise对象的两种自动Generator执行器包装成一个模块
- 使用条件：yield后只能为Thunk函数或Promise对象或Promise对象数组

#### 基于Promise的执行器

- 实现
    ```js
    function run(fn) {
      let gen = fn()
      function cb(data) {
        // 将上一个任务返回的data作为参数传给next方法，控制权交回到Generator
        //这里将result变量引用{value, done}对象，不要和Generator中的`let result = yield xxx`搞混
        let result = gen.next(data)
        if (result.done) return result.value
        result.value.then(function(data){
          // resolved之后会执行cb(data)，开启下一次循环，实现自动执行
          cb(data)
        })
      }
      cb()
    }
    ```

#### 源码分析

- 其实和上面的实现类似
    ```js
    function co(gen) {
      var ctx = this;
      var args = slice.call(arguments, 1) // 除第一个参数外的所有参数
      // 返回一个Promise对象
      return new Promise(function(resolve, reject) {
        // 如果是Generator函数，执行获取遍历器对象gen
        if (typeof gen === 'function') gen = gen.apply(ctx, args);
        if (!gen || typeof gen.next !== 'function') return resolve(gen);
        // 第一次执行遍历器对象gen的next方法获取第一个任务
        onFulfilled();
        // 每次异步任务执行完，resolved以后会调用，控制权又交还给Generator
        function onFulfilled(res) {
          var ret;
          try {
            ret = gen.next(res); // 获取{value,done}对象，控制权在这里暂时交给异步任务，执行yield后的异步任务
          } catch (e) {
            return reject(e);
          }
          next(ret); // 进入next方法
        }
        // 同理可得
        function onRejected(err) {
          var ret;
          try {
            ret = gen.throw(err);
          } catch (e) {
            return reject(e);
          }
          next(ret);
        }
        // 关键
        function next(ret) {
          // 遍历执行完异步任务后，置为resolved，并将最后value值返回
          if (ret.done) return resolve(ret.value);
          // 获取下一个异步任务，并转为Promise对象
          var value = toPromise.call(ctx, ret.value);
          // 异步任务结束后会调用onFulfilled方法（在这里为yield后的异步任务设置then的回调参数）
          if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
          return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
            + 'but the following object was passed: "' + String(ret.value) + '"'));
        }
      })
    }
    ```
    其实还是一样，为Promise对象then方法指定回调函数，在异步任务完成后出发回调函数，在回调函数中执行Generator的next方法，进入下一个异步任务，实现自动执行。
    举个🌰
    ```js
    'use strict';
    const fs = require('fs');
    const co =require('co');
    function read(filename) {
      return new Promise(function(resolve, reject) {
        fs.readFile(filename, 'utf8', function(err, res) {
          if (err) {
            return reject(err);
          }
          return resolve(res);
        });
      });
    }
    co(function *() {
      return yield read('./a.js');
    }).then(function(res){
      console.log(res);
    });
    ```

- 处理并发的异步操作
    把并发操作放数组/对象中放yield后面


## lesson4 async函数

### 语法糖

- 比较
    ```js
    function* asyncReadFile () {
      const f1 = yield readFile('/etc/fstab');
      const f2 = yield readFile('/etc/shells');
      console.log(f1.toString());
      console.log(f2.toString());
    };
    const asyncReadFile = async function () {
      const f1 = await readFile('/etc/fstab');
      const f2 = await readFile('/etc/shells');
      console.log(f1.toString());
      console.log(f2.toString());
    };
    ```
    看起来只是写法的替换，实际上有这样的区别
    - async函数内置执行器，不需要手动执行next方法，不需要引入co模块
    - async适用更广，co模块对yield后的内容严格限制为Thunk函数或Promise对象，而await后可以是Promise对象或原始类型值
    - 返回Promise，这点和co比较像

- 用法
    - async标识该函数内部有异步操作
    - 由于async函数返回的是Promise，所以可以将async函数作为await命令的参数
    - async函数可以使用在函数、方法适用的许多场景

### 语法

#### 返回的Promise

- async函数只有在所有await后的Promise执行完以后才会改变返回的Promise对象的状态（return或者抛错除外）即只有在内部操作完成以后才会执行then方法
- async函数内部return的值会作为返回的Promise的then方法回调函数的参数
- async函数内部抛出的错误会使得返回的Promise变成rejected状态，同时错误会被catch捕获

#### async命令及其后的Promise

- async命令后如果不是一个Promise对象，则会被转成一个resolved的Promise
- async命令后的Promise如果抛错了变成rejected状态或者直接rejected了，都会使得async函数的执行中断，错误可以被then方法的回调函数catch到
- 如果希望async的一个await Promise不影响到其他的await Promise，可以将这个await Promise放到一个try...catch代码块中，这样后面的依然会正常执行，也可以将多个await Promise放在一个try...catch代码块中，此外还可以加上错误重试

#### 使用注意

- 相互独立的异步任务可以改造下让其并发执行（Promise.all）
    ```js
    let [foo, bar] = await Promise.all([getFoo(), getBar()]);
    // 听说下面这种也可以？
    let fooPromise = getFoo();
    let barPromise = getBar();
    let foo = await fooPromise;
    let bar = await barPromise;
    ```

- forEach
    ```js
    forEach(async function a() {
      let m = await pro
    })
    ```
    这种写法可能不能正常工作，因为forEach时的若干个操作是并行的，应改为for循环

### 实现

- 其实就是将执行器和Generator函数封装在一起，详见上一课

### 举举🌰

- 并发请求，顺序输出
    ```js
    async function logInOrder(urls) {
      // 并发读取远程URL
      const textPromises = urls.map(async url => {
        const response = await fetch(url);
        return response.text();
      });
      // 按次序输出
      for (const textPromise of textPromises) {
        console.log(await textPromise);
      }
    }
    ```

### 异步遍历器

#### 回到过去


- Generator的next方法是同步的，需要立即返回结果，所以在处理异步操作时我们返回的实际上是{value: Promise, done: false}类似这样的对象。理解没错的话Promise对象创建时传入的函数会立即执行，异步操作例如读取文件请求就开始了。而后我们为这个Promise对象指定then方法的参数回调函数去接收异步操作完成的信号，然后在回调函数中再次调用next获取下一个{value: Promise, done: false}
- 提案：为异步操作提供原生的遍历器接口，使得value和done值都可以异步返回，称为异步遍历器

#### more

- 累了，困了，不开心看了，下次再补充
