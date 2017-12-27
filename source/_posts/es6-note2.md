---
title: ES6读书小笔记-2
tags:
  - es6
  - javaScript
categories:
  - code
  - note
abbrlink: d82f038a
date: 2017-11-06 14:12:52
---

![](http://opo02jcsr.bkt.clouddn.com/43cc41b24f8857bdc266fd2ee04ebb01.jpg)
<!-- more -->

> 本文为阅读阮一峰老师的Es6入门一书时做的笔记积累，记录下来的多为个人觉得有必要注意的点，看官老爷们随意就好。

## lesson1 Symbol

### 概述

- javaScript第七种原始数据类型Symbol
    ``let s = Symbol('foo')``
    通过Symbol函数生成，每个Symbol类型的变量值都独一无二，作为一种类似于字符串的数据结构，可以避免变量名冲突
    Symbol函数接收一个参数用于描述该Symbol实例，不影响生成的Symbol实例的值
    Symbol值不能与其他类型进行运算（模板字符串中也不可以），可以显示转为字符串和布尔值（String(), Boolean()）

- Symbol作为属性名
    - 不能用`.`，因为`.`是去取字符串对应属性名
    - 在对象中使用作为属性名时，需使用`[s]`否则也会被当做字符串

- `Symbol.for`, `Symbol.keyFor`
    ```js
    let s1 = Symbol.for("foo");
    Symbol.keyFor(s1) // "foo"
    let s2 = Symbol("foo"); // 先搜索全局，已存在该key则返回已存在的
    Symbol.keyFor(s2) // undefined
    ```

### 内置Symbol值

- `Symbol.hasInstance`
    对象的`Symbol.hasInstance`属性指向一个内部方法，其他对象使用instanceOf判断实例时，会调用这个内部方法
    ```js
    class Even {
      static [Symbol.hasInstance](obj) {
        return Number(obj) % 2 === 0;
      }
    }
    1 instanceOf Even
    ```

- `Symbol.isConcatSpreadable`
    表示该对象用于Array.prototype.concat()时是否可以展开，数组默认可展开，默认值为undefined，对象默认不可展开

- `Symbol.species`
    指向当前对象的构造函数，创造实例时会调用这个方法，即使用该属性返回的函数作为构造函数
    ```js
    static get [Symbol.species]() {
      return this;
    }
    ```

- `Symbol.match`, `Symbol.replace`, `Symbol.split`, `Symbol.search`
- `Symbol.iterator`
    指向该对象的默认遍历器方法
    对象进行for...of循环时，会调用Symbol.iterator方法，返回该对象的默认遍历器
    详见后续章节
- `Symbol.toPrimitive`
- `Symbol.toStringTag`
    指向一个方法，在该对象上调用`Object.prototype.toString()`时，如果该属性存在，则他的返回值会出现在toString方法返回的字符串之中，比如[Object Array]
    新增内置对象举个例子：``JSON[Symbol.toStringTag]：'JSON'``


## lesson2 Set和Map

### Set

#### 基本

- Set构造函数生成，成员值唯一，(判断与===区别在于NaN)，两个空对象视为不同

- 实例属性和方法
    - 属性： `Set.prototype.constructor`, `Set.prototype.size`
    - 方法： `add(value)`, `delete(value)`, `has(value)`, `clear()`
    ```js
    Array.from可以将Set转为数组
    // 数组去重
    function dedupe(array) {
      return Array.from(new Set(array));
      // return [...new Set(array)]
    }
    ```

- 遍历
    - keys(), values(), entries(), forEach() 方法
    - 遍历顺序为插入顺序，或可用于设置指定顺序的回调函数
    - Set的键名键值相同
    - Set默认可遍历，默认遍历器生成函数是values方法，这意味着，可以省略values方法，直接用for...of循环遍历 Set
    ``Set.prototype[Symbol.iterator] === Set.prototype.values``
    - `...`内部使用`for ... of`，故可以使用`[...Set]`，转为数组后可以方便使用数组方法如map和filter

#### WeakSet

- 成员只能为对象
- 弱引用，垃圾回收机制对对象引用计数时不考虑WeakSet中对对象的引用
- ``new WeakSet()`` 可以接收任何具有 `Iterable` 接口的对象作为参数，但必须注意加入`WeakSet`的成员必须为对象
- WeakSet有以下三个方法：`add(value)`, `delete(value)`, `has(value)`，没有size属性，不可遍历（没有forEach和clear方法）

### Map

#### 基本

- 键值对的集合（Hash结构），键和对象不一样，不局限于字符串。
- 任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构都可以作为Map构造函数的参数
- 只有对同一个对象的引用或者严格相等的简单类型（包括NaN）才会生成一样的Map

- 实例属性和方法
    - 属性： `Map.prototype.constructor`, `Map.prototype.size`
    - 方法： `set()`, `get()`, `delete(value)`, `has(value)`, `clear()`

- 遍历
    - 类似上述Set的遍历
    - Map 结构的默认遍历器接口（Symbol.iterator属性），就是entries方法

#### 与其他数据结构转换
- 数组，对象，JSON互转
    ```js
    function strMapToObj(strMap) {
      let obj = Object.create(null);
      for (let [k,v] of strMap) {
        obj[k] = v;
      }
      return obj;
    }
    ```

#### WeakMap

- 键名只能为对象
- WeakMap的键名所指向的对象，不计入垃圾回收机制
- WeakMap有以下三个方法：`get`, `set`, `delete(value)`, `has(value)`，没有size属性，不可遍历（没有forEach和clear方法）

![](http://opo02jcsr.bkt.clouddn.com/a3ca4dc0aad89c377838dcf51642e109.jpg)


## lesson3 Proxy

### 观察

- 举个栗子
    当你为对象a赋值a.b=c时，你希望在b属性赋值时有一个范围大小的校验，超出范围抛错，这个时候我们可能会想到重载set方法,比如：
    ```js
    let a = {}
    Object.defineProperty(a, 'b', {
      set(x) {
        if (x>100) {
          throw new RangeError('invalid range')
        }
        this.b = x
      }
    })
    ```
    动手以后发现一个问题...这样会栈溢出，因为在set内再set了b的值，无限循环...变通一下：
    ```js
    let a = {}
    Object.defineProperty(a, 'b', {
      get(x) {
        return this.c
      }
      set(x) {
        if (x>100) {
          throw new RangeError('invalid range')
        }
        this.c = x
      }
    })
    ```
    然而总要这么写感觉很麻烦，而且如果是对一类属性进行操作时，重复写很没必要，换用Proxy写法：
    ```js
    let a = {}
    let handler = {
      set(obj, prop, value, receiver) {
        if (prop === 'b') {
          if (value>100) {
            throw new RangeError('invalid range')
          }
        }
        obj[prop] = value
      }
    }
    let proxy = new Proxy(a, handler)
    ```
    看起来也舒服多了，而且可以根据属性名在set方法内做判断，更可扩展

### 庖丁解牛

- 代理proxy
    ```js
    let target = {};
    let handler = {};
    let proxy = new Proxy(target, handler);
    // 将代理的所有内部方法转发至目标
    proxy.a = 1 => target.a = 1;
    target.b = 4 => proxy.b = 4;
    target !== proxy
    target.__proto__ === proxy.__proto__
    // 应在代理对象上操作，代理才能生效
    handler = {get(){return 12}}
    target.v // undefined
    proxy.v // 12
    ```

- Proxy支持的拦截操作
    ```js
    get(target, propKey, receiver) // proxy.foo, proxy['foo']
    set(target, propKey, value, receiver) //proxy.foo = v, proxy['foo'] = v
    has(target, propKey) // propKey in proxy
    deleteProperty(target, propKey) // delete proxy[propKey]
    ownKeys(target) // Object.getOwnPropertyNames(proxy), Object.getOwnPropertySymbols(proxy), Object.keys(proxy)
    getOwnPropertyDescriptor(target, propKey) // Object.getOwnPropertyDescriptor(proxy, propKey)
    defineProperty(target, propKey, propDesc) // Object.defineProperty(proxy, propKey, propDesc), Object.defineProperties(proxy, propDescs)
    preventExtensions(target) // Object.preventExtensions(proxy)
    getPrototypeOf(target) // Object.getPrototypeOf(proxy)
    isExtensible(target) // Object.isExtensible(proxy)
    setPrototypeOf(target, proto) // Object.setPrototypeOf(proxy, proto)
    apply(target, object, args) // 拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)
    construct(target, args) // new proxy(...args)
    ```

- 代理句柄handler
    句柄对象的方法可以复写代理的内部方法，具体为上述的14种。
    - 举个🌰
    ```js
    function Tree() {
      return new Proxy({}, handler);
    }
    var handler = {
      get: function (target, key, receiver) {
        if (!(key in target)) {
          target[key] = Tree();  // 自动创建一个子树
        }
        return Reflect.get(target, key, receiver);
      }
    }
    var tree = new Tree()
    tree.branch1.branch2.twig = "green"
    ```
    - 再来个🌰
    ```js
    // 实现对in操作符隐藏属性
    var handler = {
      has (target, key) {
        if (key[0] === '_') {
          return false;
        }
        return key in target;
      }
    };
    var target = { _prop: 'foo', prop: 'foo' };
    var proxy = new Proxy(target, handler);
    '_prop' in proxy // false
    ```
    - 特别注意
        如果目标对象不可扩展或者目标对象的属性不可写或者不可配置时，代理不能生效，可能会报错
        需注意一些特定的方法对返回值有要求，不如重写isExtensible方法时，返回值与目标对象的isExtensible属性应一致，否则会报错
        利用代理重写可以做很多事情比如隐藏属性、对某些属性、操作符屏蔽、拦截内在方法并且加上自己想要的逻辑处理去得到预期结果等

### 饭后甜点

- Proxy.revocable
    返回一个对象，proxy属性对应Proxy实例，revoke属性为revoke方法可以取消Proxy实例
    ```js
    let {proxy, revoke} = Proxy.revocable(target, handler);
    proxy.foo = 1
    revoke()
    proxy.foo // TypeError: Revoked
    ```

- this问题
    - 代理以后目标对象内部的this指向的是Proxy实例而不是目标对象
    - 有时候可能因为this指向问题导致代理达不到预期效果
    ```js
    // jane的name属性实际存储在外部的WeakMap对象的_name上，导致后续取不到值
    const _name = new WeakMap();
    class Person {
      constructor(name) {
        _name.set(this, name);
      }
      get name() {
        return _name.get(this);
      }
    }
    const jane = new Person('Jane');
    jane.name // 'Jane'
    const proxy = new Proxy(jane, {});
    proxy.name // undefined
    ```
    - 某些原生对象的部分属性需要this指向原生对象时才能获取，如Date.getDate()，此时proxy get时需要注意this绑定原始对象
    ```js
    const target = new Date('2015-01-01');
    const handler = {
      get(target, prop) {
        if (prop === 'getDate') {
          return target.getDate.bind(target);
        }
        return Reflect.get(target, prop);
      }
    };
    const proxy = new Proxy(target, handler);
    proxy.getDate() // 1
    ```

### 进阶

- TO BE CONTINUED!


## lesson4 Reflect

### 初识

- Reflect对象与Proxy对象一样，是为了操作对象而提供的新API，存在的原因如下：
    - 将Object对象的一些内部方法添加到Reflect对象上，且以后的新方法都部署到Reflect对象上，完成分离
    - 让对象操作变成函数行为
    - 修改Object对象一些内部方法在出错时的返回
    - Proxy覆写对象方法时，提供一个Reflect对象用来获取原始方法，以设置默认值，再此基础上再做功能添加和修改

### 揭面

- 静态方法
    对应于Proxy可覆写的方法，有13个静态方法

- 注意
    - Proxy和Reflect联用的时候要小心，可能一个拦截会触发另一个拦截
    ```js
    let p = {
      a: 'a'
    };
    let handler = {
      set(target, key, value, receiver) {
        console.log('set');
        Reflect.set(target, key, value, receiver)
      },
      defineProperty(target, key, attribute) {
        console.log('defineProperty');
        Reflect.defineProperty(target, key, attribute);
      }
    };
    let obj = new Proxy(p, handler);
    obj.a = 'A';
    // set
    // defineProperty
    ```
        在Reflect.set传入receiver的时候触发了Proxy.defineProperty，不传入receiver时不会触发defineProperty拦截

    - 对于参数的要求、转换和报错处理

### 🌰

- 使用Proxy实现观察者模式
    ```js
    const person = observable({
      name: '张三',
      age: 20
    });
    function print() {
      console.log(`${person.name}, ${person.age}`)
    }
    observe(print);
    person.name = '李四';
    // 输出
    // 李四, 20
    /**************************/
    const queuedObservers = new Set();
    const observe = fn => queuedObservers.add(fn);
    const observable = obj => new Proxy(obj, {set});
    function set(target, key, value, receiver) {
      const result = Reflect.set(target, key, value, receiver);
      queuedObservers.forEach(observer => observer());
      return result;
    }
    ```

## lesson5 遍历器Iterator

### 遇见

- why Iterator
    js中数据集合的概念越来越多，如果能有一种统一的访问方式将是极好的。Iterator的设计就基于此，通过为相应数据结构部署iterator接口让该数据结构可通过统一的方式:for...of遍历

- 遍历过程：
    - 创建一个指针对象指向当前数据结构的初始位置（遍历器对象实际为一个指针对象）
    - 调用指针对象的next方法，直到指向数据结构的结束位置
    ```js
    // 遍历器生成函数
    function makeIterator(array) {
      var nextIndex = 0;
      return {
        next: function() {
          return nextIndex < array.length ?
            {value: array[nextIndex++]} :
            {done: true};
        }
      };
    }
    ```

- 一种数据结构，只要部署了Iterator接口，就视为可遍历的

### 相识

#### 默认Iterator接口

- 默认的Iterator接口部署在[Symbol.iterator]属性上，`Symbol.iterator`属性键为Symbol对象，值为一个函数，即遍历器生成函数，执行该函数会返回一个遍历器对象，该对象具有一个next方法，调用该方法可以返回{value, done}对象，代表了当前成员的信息
- 部分数据结构如Array、Set、Map、String等已经部署了Iterator接口，对象则需要手动添加这样的方法实现Iterator接口
- 对于非线性数据结构，Iterator接口实际上就是一种线性转换，下例为class实现遍历器
    ```js
    class RangeIterator {
      constructor(start, stop) {
        this.value = start;
        this.stop = stop;
      }
      [Symbol.iterator]() { return this; }
      next() {
        var value = this.value;
        if (value < this.stop) {
          this.value++;
          return {done: false, value: value};
        }
        return {done: true, value: undefined};
      }
    }
    function range(start, stop) {
      return new RangeIterator(start, stop);
    }
    for (var value of range(0, 3)) {
      console.log(value); // 0, 1, 2
    }
    ```
#### 举个🌰

- 实现指针
    ```js
    function Node(value) {
      this.value = value;
      this.next = null
    }
    // for...of时会调用改遍历器生成函数
    Node.prototype[Symbol.iterator]= function() {
      // 返回的遍历器对象
      var iterator = {
        next: next
      }
      // 当前成员
      var current = this
      next() {
        if(current) {
          var value = current.value;
          // 移动指针
          current = current.next;
          return { done: false, value: value };
        }
        return { done: true, value: undefined };
      }
      return iterator
    }
    // 新建对象，因为在原型上实现的遍历器生成函数，所以每个实例都实现了遍历器接口
    var one = new Node(1);
    var two = new Node(2);
    var three = new Node(3);
    // 当前成员的next指向下一个成员，在next方法中实现指针移动
    one.next = two;
    two.next = three;
    // 对对象使用for...of时，去查找[Symbol.iterator]属性，找到后循环调用next方法，直到返回值得done属性为true
    for (var i of one){
      console.log(i); // 1, 2, 3
    }
    ```

- 如果`Symbol.iterator`方法对应的不是遍历器生成函数，则会报错

### 在何方

#### 调用场合

- 解构赋值
- 扩展运算符
    任何实现了Iterator接口（可遍历）的数据结构都可以通过`...`将其转化为数组

- `yield*`后跟一个可遍历数据结构时，会调用该结构的遍历器接口
- `for...of`, `Array.from`, `Map`, `Set`, `Promise.all()`, `Promise.race()`

#### 字符串、数组等的遍历器

- 字符串
    for...of能够正确识别32位UTF-16字符
    ```js
    var someString = "hi";
    typeof someString[Symbol.iterator]
    // "function"
    var iterator = someString[Symbol.iterator]();
    // 当然你也可以修改Symbol.iterator方法达到你想要的遍历结果
    iterator.next()  // { value: "h", done: false }
    iterator.next()  // { value: "i", done: false }
    iterator.next()  // { value: undefined, done: true }
    ```

- return, throw方法
    这两个方法都是在设置遍历器生成函数时可选的，一般配合generator使用，所以下次再说

- 数组
    - for ... of 只返回具有数字索引的键值的值
    - 类数组对象可以使用数组的默认遍历生成器达到遍历效果（要求是数字索引以及具有length属性）

- Map, Set
    ```js
    // Map遍历返回的是数组[k, v]，Set返回的是值
    for (let [key, value] of map) {
      console.log(key + ' : ' + value);
    }
    ```

- 类数组对象
    利用Array.from将其转化为数组，再使用数组的遍历器接口用for...of实现遍历

#### for ... of 注意

- 可以结合break、continue、return使用
- 提供了多种数据结构的统一访问方式
- 相比for ... in，后者遍历的是键，且键名为字符串，还可能会遍历原型上的键
