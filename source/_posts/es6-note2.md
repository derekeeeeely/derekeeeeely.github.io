---
title: ES6读书笔记-2 解构
tags:
  - es6
  - javaScript
categories: note
abbrlink: 78048d04
date: 2017-11-03 13:44:06
---
![](http://opo02jcsr.bkt.clouddn.com/588efc8d85be9ccfed074cd6f9eba69b.jpg)
<!-- more -->

# lesson3 解构

- 数组解构的本质是`模式匹配`，只要某种数据结构具有`Iterator`接口，就都可以采用数组形式解构。egs: Array, Set, Generator.

```js
// 解构赋值设置默认值是根据右侧是否===undefined来决定右侧是否有值，有则覆盖默认值
let [i = 1] = [] // 1
let [i = 1] = [undefined] // 1
let [i = 1] = [null] // null
// 下例可以看出默认值先设置，再进行解构赋值
let [x = 1, y = x] = [1, 2]; // x=1; y=2
```

> ``[a, b, c] = arr``  equals   ``a = arr[0]  b = arr[1]``


- 对象的解构赋值需要变量与对象属性名相同，本质是先找到同名的属性，然后把值赋给对应的变量

```js
// foo是匹配模式，不是变量，sss是被赋值的变量
let { foo: sss, bar: ddd } = { foo: "aaa", bar: "bbb" };
foo // foo is not defined
sss // "aaa"
// 默认值生效的条件是，对象的属性值严格等于undefined，结构失败为undefined
let { x = 2 } = { x: null }
x// null
// 如果解构模式是嵌套的对象，父属性不存在时会报错
let {foo: {bar}} = {baz: 'baz'}
// 理解下面的为三次解构 node.loc node.loc.start node.loc.start.line
let { loc, loc: { start }, loc: { start: { line }} } = node
```

> 原来项目里天天写的``const {a, b} = this.state``是解构赋值！

> ``let {a:c, b:d} = obj`` equals ``c = obj.a d = obj.b``

- 字符串解构赋值

```js
let [a,b,c] = 'skt'
let {length: len} = 'skt'
```

- 数值、布尔值、null、undefined

```js
// 等号右边先转为对象再进行解构赋值，故undefined和null会报错
let {toString: s} = 123;
s === Number.prototype.toString // true
```

- 用途

  - 交换变量值
  ```js
  [x, y] = [y, x]
  ```
  - 从函数返回多个值
  - 函数参数的定义、默认值
  ```js
    // 参数是一组有次序的值
    function f([x, y, z]) { ... }
    f([1, 2, 3]);
    // 参数是一组无次序的值
    function f({x, y, z}) { ... }
    f({z: 3, y: 2, x: 1});
  ```
  - 提取JSON数据
  ```js
  let { id, status, data: number } = jsonData;
  ```
  - 遍历map
  ```js
  // 获取key、value
  for (let [key, value] of map) {
    console.log(key + " is " + value);
  }
  ```
  - 获取输入模块的指定方法

  ```js
  const { SourceMapConsumer, SourceNode } = require("source-map");
  ```
