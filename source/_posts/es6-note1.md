---
title: ES6读书小笔记-1
tags:
  - es6
  - javaScript
categories:
  - code
  - note
abbrlink: 631da001
date: 2017-10-31 02:17:11
---

![](http://opo02jcsr.bkt.clouddn.com/79a64d61c2d6c28e0c0272ed0c1a5b1a.jpg)
<!-- more -->

> 本文为阅读阮一峰老师的Es6入门一书时做的笔记积累，记录下来的多为个人觉得有必要注意的点，看官老爷们随意就好。

## lesson1 introduce

### 新的语法成为标准的过程

- stage
  ```bash
  Stage 0 - Strawman（展示阶段）
  Stage 1 - Proposal（征求意见阶段）
  Stage 2 - Draft（草案阶段）
  Stage 3 - Candidate（候选人阶段）
  Stage 4 - Finished（定案阶段）
  ```

### babel

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


## lesson2 let/const

### TDZ

  只在声明的块级作用域域内有效、不可重复声明、TDZ

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

### const

- freeze
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

## lesson3 解构

### 数组解构

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

### 对象解构

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

### 字符串、数值、布尔值、null、undefined

- 如下
```js
let [a,b,c] = 'skt'
let {length: len} = 'skt'
```

- 如下
```js
// 等号右边先转为对象再进行解构赋值，故undefined和null会报错
let {toString: s} = 123;
s === Number.prototype.toString // true
```

### 用途

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

## lesson4 字符串扩展

- unicode表示
  js中，可以用\uxxxx表示字符，xxxx为该字符的Unicode码位。js内部，字符以UTF-16格式存储，即每个字符为2个字节(0x0000-0xFFFF)，对于需要4字节存储的例如汉字，在处理时可能会出现误判。
- `codePointAt`
  ```js
  // 判断是否为4字节组成
  function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
  }
  ```
- `String.fromCodePoint`
    返回码点对应字符，支持4字节，即32位UTF-16字符
- 字符串遍历
    ``for ... of ...``遍历字符串，可以做到支持32位UTF-16字符
- 提案 `at`
    charAt返回的是UTF-16字符的前两个字节，提案at可以支持4字节UTF-16字符，需要垫片库支持
- ``include(), startsWith(), endsWith()``
    字面理解，注意可以传一个下标参数，表示查找位置
- ``repeat(), padStart(5, s = ''), padEnd(5, e = '')``
    padStart填充在前，padEnd填充在后，若原字符串长度大于第一个参数，则不填充返回原字符串，若加起来大于第一个参数则保留原，去除需添加的多余的。
    ```js
    '123456'.padStart(10, '0') // "0000123456"
    '12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
    ```
- 模板字符串
- 标签模板
    ```js
    let total = 30;
    let msg = passthru`The total is ${total} (${total*1.05} with tax)`;
    // ['The total is ', ' ', ' (', ' with tax)'], 30, 31.5
    function passthru(literals, ...values) {
    let output = "";
    let index;
    for (index = 0; index < values.length; index++) {
        output += literals[index] + values[index];
    }
    output += literals[index]
    return output;
    }
    msg // "The total is 30 (31.5 with tax)"
    ```

    > 过滤字符串

    ```js
    let message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

    function SaferHTML(templateData) {
    let s = templateData[0];
    for (let i = 1; i < arguments.length; i++) {
        let arg = String(arguments[i]);

        // Escape special characters in the substitution.
        s += arg.replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;");

        // Don't escape special characters in the template.
        s += templateData[i];
    }
    return s;
    }
    ```

    > 其他用途有：i18n、模板处理、引入其他语言等

    ```js
    // 下面的hashTemplate函数
    // 是一个自定义的模板处理函数
    let libraryHtml = hashTemplate`
    <ul>
        #for book in ${myBooks}
        <li><i>#{book.title}</i> by #{book.author}</li>
        #end
    </ul>
    `;
    ```

## lesson5 数值扩展

- 0o/0O 八进制 0b/0B 二进制
- `Number.isFinite` `Number.isNaN`
    与全局`isFinite`、`isNaN`的差别在于前者只对数值有效，非数值一律返回`false`后者是先将非数值转化为数值再判断。
- `Number.ParseInt` `Number.ParseFloat` `Number.isInteger`
    逐步减少全局性方法，使得语言逐步模块化
- `Number.EPSILON` 可以接受的最小误差范围
- `isSafeInteger`
    ```js
    Number.isSafeInteger(9007199254740993)
    // false
    Number.isSafeInteger(990)
    // true
    Number.isSafeInteger(9007199254740993 - 990)
    // true
    9007199254740993 - 990
    // 返回结果 9007199254740002
    // 正确答案应该是 9007199254740003
    ```
    9007199254740993不是一个安全整数，但是Number.isSafeInteger会返回结果，显示计算结果是安全的。这是因为，这个数超出了精度范围，导致在计算机内部，以9007199254740992的形式储存
- Math对象扩展
    - 指数运算符`**`
    - `提案` Integer

## lesson6 函数扩展

### 参数设置默认值、rest参数

- basic
- with解构
    ```js
    function m1({x = 0, y = 0} = {}) {
        return [x, y];
    }
    function m2({x, y} = { x: 0, y: 0 }) {
        return [x, y];
    }
    ```
    理解二者区别即可，其余可参见[解构](http://www.derekz.cn/posts/78048d04/)

- 函数的`length`属性：未指定默认值的参数个数
- 参数一旦设置了默认值，函数在声明初始化时会形成单独的作用域，初始化结束后释放掉。
    ```js
    let x = 1
    function m(y=x){
        let x = 2
        console.log(y)
    }
    m()// 1
    // 如果全局x不存在会报错，第二行相当于发生了let y = x, x指向全局的x, 不受内部x影响

    var x = 1;
    function foo(x, y = function() { x = 2; }) {
    var x = 3;
    y();
    console.log(x);
    }
    // 理解：这里有三个作用域，三个x，各不相干 3
    // 去掉var以后参数中的两个x和函数体内x是一样的，外部还是不变 2
    ```

- 应用
    指定参数必传，否则报错，即给参数赋值一个默认的函数抛错的结果，没传时得到的是这个抛出的错
    用默认赋值undefined表示该参数可省略
- 函数的length不包含rest参数
- rest参数之后不能再有参数
- 只要参数使用了默认值、解构赋值、或者扩展运算符，就不能显式指定严格模式

### 箭头函数

- 注意
    - 箭头函数体内的this指向函数定义时的对象，不是对应运行时的对象
    - 不可以当做构造函数
    - 不可以使用yield
    - 不可以使用arguments
    - 不可以使用bind、call、apply来改变箭头函数this指向

- this指向的固定化，并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this
  ```js
  const headAndTail = (head, ...tail) => [head, tail];
  headAndTail(1, 2, 3, 4, 5) // [1,[2,3,4,5]]
  ```

### 其他

- 双冒号运算符
  ```js
  obj::func === func.bind(obj)
  foo::bar(...arguments) === bar.apply(foo, arguments)
  ```

- 尾调用优化
  ```js
  // 函数f的最后一步为函数g调用
  function f(x){
    return g(x);
  }
  ```
  函数调用会在内存在生成调用记录，即调用帧，如A函数中调用B函数，则会在A的调用帧上方形成B的调用帧，B调用完后，返回至A才会消失。尾调用实际上已经不用保存外层A的调用帧了，可以优化。
  **只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”**

- 尾递归优化
  ```js
  // 非尾递归
  function Fibonacci (n) {
    if ( n <= 1 ) {return 1};
    return Fibonacci(n - 1) + Fibonacci(n - 2);
  }
  // 尾递归
  function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
    if( n <= 1 ) {return ac2};
    return Fibonacci2 (n - 1, ac2, ac1 + ac2);
  }

  传统递归需要保存相当多的调用帧，尾递归只保存一个，节省内存，避免发生栈溢出
  ```

- 柯里化
    将多参数函数变成单参数形式（具体下次再论）
- es2017支持函数最后一个参数尾逗号
- 在不需要错误实例的时候，提案try...catch...菜单catch块不需要参数

## lesson7 数组扩展

### 扩展运算符

- `...` 可以理解为rest参数的逆运算，将数组转化为逗号分隔的参数序列，如``Math.max(...[14, 3, 77])``
- 复制数组（直接赋值是引用，指向相同地址）
    ```js
    // es5
    a1 = [1,2];    a2 = a1.concat()
    // es6
    a1 = [1,2];    a2 = [...a1]
    ```

- 合并数组
- 与解构结合
    扩展运算符用于数组赋值时，只能作为最后一个参数
    ```js
    const [a, ...b] = [1,2,3,4]
    ```

- 字符串
    ```js
    // 生成数组
    [...'hello']
    // 返回字符串正确长度，兼容4字节UTF-16编码的Unicode字符
    function length(str) {
    return [...str].length;
    }
    ```

- 实现了 Iterator 接口的对象（下回分解）
- Map Set Generator（下回分解）

### Array方法

- `Array.from()`
     将`arraylike`对象和可遍历对象转为数组
    ```js
    // Array.prototype.slice
    Array.from
    // Array.from(arrayLike).map(x => x * x)
    // 接收第二个参数，类似map，可用于如NodeList对象等处理，获取dom节点的属性
    Array.from(arrayLike, x => x * x)
    Array.from([1, , 2, , 3], (n) => n || 0)
    // 第三个参数，可以绑定this
    ```
     `...`调用的是遍历器接口（`Symbol.iterator`），若对象没有部署这个接口，就不能转化为数组，`Array.from`可以转换任何有length属性的对象

     因吹斯听
    ```js
    Array.from({ length: 2 }, () => 'jack')
    效果：第一个参数为第二个参数（函数）指定执行次数
    ```

- `Array.of`
     总是返回参数值组成的数组，如果没有参数，就返回一个空数组
    ```js
    // 弥补构造函数Array()的不足（1-2个参数时）
    function ArrayOf(){
        return [].slice.call(arguments);
    }
    ```
- `copyWithin` 用途不明
- `find` `findIndex` 前者返回第一个值/`undefined`，后者返回位置/-1
- `fill` 数组填充，可以指定起始、结束位置
- `keys()` `values()` `entries()` 用于遍历数组，返回遍历器对象，可以使用for...of循环也可以手动调用遍历器对象的next方法。分别对应 键、值、键值对（[0,'a']）
- `includes()` 判断数组中是否包含某个值，解决了indexOf对NaN的误判（因为NaN!==NaN，indexOf是===判断）
- 空位，es6上述数组扩展对空位处理比较一致，值设为undefined，遍历时视为存在


## lesson8 对象扩展

### 写法

- 属性、方法简写
    ```js
    { a } { a() {} }
    ```
- 属性名表达式、方法名表达式
    ```js
    { ['a' + 'bc']: 'a' }
    { ['a' + 'bc']() {} }
    // 注意属性名为对象时，会自动转为字符串`[object Object]`
    ```
- 方法名name
    ```js
    // 这种情况下`obj.foo.name`会报错
    const obj = {
        get foo() {},
        set foo(x) {}
    };
    // 正确写法
    const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');
    descriptor.get.name // get foo
    // special
    (new Function()).name // "anonymous"
    doSomething.bind().name // "bound doSomething"
    ```
### 方法

#### `Object.is`

- 比较
    ```js
    // 除了下列情况外和===的表现一致
    +0 === -0 //true
    NaN === NaN // false
    Object.is(+0, -0) // false
    Object.is(NaN, NaN) // true
    ```
#### `Object.assign`

- 用于对象合并，将源对象的所有可枚举属性复制到目标对象
    ```js
    Object.assign(target, source1, source2);
    ```

- 注意
    非对象出现在第一个参数会被转为对象，出现在后面只有字符串会以字符数组形式参与合并。（因为只有字符串的包装对象，会产生可枚举属性）
    对于undefined和null无法转为对象，多以出现在第一个参数位会报错，出现在后面会被忽略。
    浅拷贝，即如果源对象的某个属性是对象，则目标对象拷贝的是该对象的引用

- 用途
    - 为对象添加属性、方法
    - 克隆对象
    ```js
    // 只克隆原有可枚举属性
    function clone(origin) {
        return Object.assign({}, origin);
    }
    // 保持继承链
    function clone(origin) {
        let originProto = Object.getPrototypeOf(origin);
        return Object.assign(Object.create(originProto), origin);
    }
    ```
    - 合并多个对象
    - 为属性设置默认值（利用覆盖，注意应为简单类型）

#### `Object.getOwnPropertyDescriptors`

- 返回指向对象的所有自身属性的描述对象
- 由于Object.assign是赋值拷贝，所以无法正确拷贝对象的get和set方法，所以需要有方式去获取描述对象，再结合defineProperties进行
- 结合Object.create使用，浅拷贝

#### `Object.getPropertyOf`, `Object.setPropertyOf`, `__proto__`

- `__proto__`
    ```js
    obj = Object.create(someOtherObj);
    obj.__proto__ = someOtherObj;
    ```

- `Object.setPrototypeOf(obj, proto)`
    将proto对象设为obj的原型，第一个参数为undefined和null时报错，非对象时自动转

- `Object.getPrototypeOf(obj)`

#### `super`

- 指向当前对象的原型对象，只有对象的方法的简写模式中有效，否则报错
- `super.foo`等同于`Object.getPrototypeOf(this).foo`

#### `Object.keys`, `Object.values`, `Object.entries`

- `Object.entries` 用途：``new Map(Object.entries(obj))``


### 属性的可枚举和遍历

#### 可枚举

- 获取属性的描述对象
    ```js
    let obj = { foo: 123 };
    Object.getOwnPropertyDescriptor(obj, 'foo')
    //  {
    //    value: 123,
    //    writable: true,
    //    enumerable: true,
    //    configurable: true
    //  }
    ```
- 忽略 enumerable为false的操作
    ```js
    for ... in // 会返回继承的属性
    Object.keys()
    JSON.stringify()
    Object.assign()
    ```
- 遍历
    ```js
    for ... in // 遍历自身和继承的**可枚举**属性，不包含Symbol属性
    Object.keys // 返回一个数组，只包含自身所有**可枚举**属性，不包含Symbol属性
    Object.getOwnPropertyNames(obj) // 返回一个数组，包含对象自身的**所有**属性，不包含Symbol属性
    Object.getOwnPropertySymbols(obj) // 返回一个数组，包含对象本身的所有Symbol属性
    Reflect.ownKeys(obj) // 返回一个数组，包含自身**所有键名**
    // 共性：遍历顺序为 数值键-字符串键-Symbol属性
    ```

### 扩展运算符

### Null传导运算符（提案）

- ``obj?.prop, obj?.[expr], func?.(...args), new C?.(...args)``
