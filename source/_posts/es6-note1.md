---
title: ES6读书笔记-1 let const
tags:
  - es6
  - javaScript
categories: note
top: 99
abbrlink: 631da001
date: 2017-10-31 02:17:11
---

![](http://opo02jcsr.bkt.clouddn.com/79a64d61c2d6c28e0c0272ed0c1a5b1a.jpg)
<!-- more -->

## lesson1

### introduce

- 新的语法成为标准的过程
```bash
Stage 0 - Strawman（展示阶段）
Stage 1 - Proposal（征求意见阶段）
Stage 2 - Draft（草案阶段）
Stage 3 - Candidate（候选人阶段）
Stage 4 - Finished（定案阶段）
```

- .babelrc
```bash
npm install --save-dev babel-preset-latest
{
    presets: [
        "latest",
        "react",
        "stage-2"
    ],
    plugins: []
}
# presets的值集有latest，react，stage-0 to stage-3（对应不同阶段的提案，选择一个）
```

- babel-cli
```bash
# -s生成sourcemap，-o指定文件，-d指定目录
babel example.js -o compiled.js -s
# babel-node提供支持ES6的REPL环境，目标文件不需要考虑转码
babel-node es6.js
```

- babel-register
```bash
提供一个钩子，在require加载`.js`, `.jsx`, `.es`, `.es6`后缀的文件时先用babel实时转码，需在require别的文件前先require该模块
require("babel-register");
require("./index.js");
```

- babel-core
```bash
需要调用babel的api时，需要require该模块
var babel = require('babel-core');
// 字符串转码
babel.transform('code();', options);
```

- babel-polyfill
```bash
默认情况下babel只转句法，不转一些es6新的对象以及方法，这种情况下需要在脚本头部引入该模块转换使其可用
import 'babel-polyfill';
```

- Traceur
```bash
google提供的babel替代品
```


## lesson2

### let和const

只在声明的块级作用域域内有效、不可重复声明、DTZ

- 循环体和循环条件中的i有单独的作用域
```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i); //输出3次
}
```

- TDZ
```js
// 在该区域内let/const定义的变量，只有等到声明以后才可以获取/使用
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError
  let tmp; // TDZ结束
  console.log(tmp); // undefined
  tmp = 123;
  console.log(tmp); // 123
}
```

> 注意：带来的问题是typeof不再绝对安全

### 块级作用域

- 没有块级作用域的问题
```js
// 场景1
var tmp = new Date();
function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}
f(); // undefined因为变量提升，所以在函数执行的上下文中找到了tmp的定义，但是没有完成赋值，所以是undefined
// 场景2 典型的for循环将i变成了全局变量
```

- 块级作用域

  - 告别丑陋的IIFE

  - 对于es6的浏览器环境还说，函数声明类似于var，可以理解成先函数变量提升``var f = undefined``，然后再继续后面，所以以下代码是会报错的
    ```js
    function f() { console.log('I am outside!'); }
    (function () {
        if (false) {
            // 重复声明一次函数f
            function f() { console.log('I am inside!'); }
        }
        f();
    }());
    // Uncaught TypeError: f is not a function
    ```

  - do提案
  ```js
  在代码块前加do，返回最后执行表达式的值
  let x = do {
    let t = f();
    t * t + 1;
  };
  ```

- const

    对于复杂类型变量，const保证的是指向该变量的内存空间的地址是不变的，变量若是可写的是可以修改变量的属性值的。
    要想冻结变量，需使用`Object.freeze()`冻结对象及对象所有属性，如下：
    ```js
    var constantize = (obj) => {
    Object.freeze(obj);
        Object.keys(obj).forEach( (key, i) => {
            if ( typeof obj[key] === 'object' ) {
            constantize( obj[key] );
            }
        });
    };
    ```

- var function let const class
    前两者声明的全局变量依然是顶层对象（window/global）的属性，后三者不再是，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩

### 提案

- 考虑到不同环境下顶层对象不同，提案在语言标准层面引入`global`对象作为顶层对象，即所有环境下global对象都是存在的，如下
```js
// 将顶层对象放入变量global中
// CommonJS 的写法
var global = require('system.global')();

// ES6 模块的写法
import getGlobal from 'system.global';
const global = getGlobal();
```
