---
title: diary
tags: diary
categories: diary
abbrlink: 917bede2
date: 2017-11-23 19:33:31
---

![](http://opo02jcsr.bkt.clouddn.com/d80db8259a7b22845d8b0612e5ed633f.jpg)
<!-- more -->

## 渣渣前端学习笔记

### React

#### setState

- key point
  - setState不会立刻更改state
  - setState通过引发一次组件更新过程来引发重新绘制
  - 多个setState效果会merge

- 同步/异步更新state
  - 由React引发的事件处理会在调用事件处理函数前会调用`batchedUpdates`将`isBatchingUpdates`置为true，使得setState不同步更新组件的state
    ![](http://opo02jcsr.bkt.clouddn.com/fe72f872acdefd741d6483979bdf3310.jpg)

  - 绕过React事件处理的情况，比如`addEventListener`以及`setTimeout`的方式可以实现setState同步更新state
  - 函数式setState也可以做到同步更新组件state
    ```js
    // 函数式setState，函数接收两个参数，返回一个新的state，多次调用将接收上一次的state后同步执行
    upfunc(state, props) {
      // do something
      return mergedState
    }
    // 函数式setState和传统setState混用时要小心
    this.setState(upfunc)
    this.setState(upfunc)
    ```

#### 装饰

- 装饰函数
  ```js
  function Person(name){
    this.name = name
  }
  Person.prototype.sayHello = function(){
    console.log(`hello ${this.name}`)
  }
  // 比如我在不改上面源码的前提下，想在sayHello的时候输出"I'm Derek, hello xxx"
  // maybe this
  function dec(f, ...arg){
    console.log("I'm Derek, ")
    typeOf f === 'Function' && f(...arg)
  }
  const Mick = new Person('Mick')
  Mick.greet = dec(Mick.greet)
  Mick.greet()
  // maybe this
  function Dec(f){
    const newPerson = Object.create(f)
    newPerson.greet = function(){
      console.log("I'm Derek, ")
      f.greet()
    }
    return newPerson
  }
  const newMick = Dec(Mick)
  newMick.greet()
  ```



- reinforce component
  - HOC
  ```js
  // accept a Component and return a new Component
  function HOC(WrappedComponent) {
    return class Wrap extends React.Component {
      render() {
        <div>do something</div>
        return <WrappedComponent {...this.props}/>
      }
    }
  }
  ```
  - render props
  ```js
  // 增强组件，将state抛出
  class Mouse extends React.Component {
    static propTypes = {
      render: PropTypes.func.isRequired
    }
    //
    state = { x: 0, y: 0 }
    //
    handleMouseMove = (event) => {
      this.setState({
        x: event.clientX,
        y: event.clientY
      })
    }
    //
    render() {
      return (
        <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>
          {this.props.render(this.state)}
        </div>
      )
    }
  }
  // 需要被增强的组件App
  const App = React.createClass({
    render() {
      return (
        <div style={{ height: '100%' }}>
          <Mouse render={({ x, y }) => (
            // The render prop gives us the state we need
            // to render whatever we want here.
            <h1>The mouse position is ({x}, {y})</h1>
          )}/>
        </div>
      )
    }
  })
  ```
  key point: `{this.props.render(this.state)}`调用包装组件Mouse的render prop(函数)，将Mouse组件的state作为参数，函数返回被包装的App组件


### js大法

#### 异步

- 主线程执行同步任务，触发异步任务时，开启另一个工作线程和主线程一起并行执行，异步任务有结果后再事件队列中放置回调函数。等待主线程当前事件循环结束后去读取事件队列，区分microtask和marcotask判断是加在当前循环尾部还是下一个循环头部。`ajax`, `fs`, `setTimeout`等都是如此。
  ![](http://opo02jcsr.bkt.clouddn.com/ee2459f5db2bcf88379fc67541e274fc.jpg)

- js代码单线程执行，通过运行环境的机制去创建工作线程实现任务的并行处理，通过回调实现控制权的交替，也即call sth now, do else later的异步效果

- 异步编程
  - 举个🌰
  ```js
  // 下例为串行异步任务的情况，可以并行的情况下for循环改写
  (function a (i, total, cb){
    if(i<total){
        b(i, function(){
            a(++i) // b(i)执行有了结果后，执行回调，下一个异步任务开启
        })
    } else {
        cb()
    }
  }(0, 10, function(){
      // 任务都执行完回调
  }))
  ```
  - 异步编程依托于回调来实现，而使用回调不一定就是异步编程

### typescript

#### javaScript的超集

- 类型声明和检查
  ```js
  // basic
  :boolean, :number, :string
  // Array
  :number[] :Array<number>
  :ReadonlyArray<number>
  // Tuple
  :[string, number]
  // Enum
  enum Color {Red = 1, Green, Blue}
  let c: Color = Color.Green
  // Any, Void, Never
  :any, :void, :never
  ```

- interface
  - 不需要显示声明某对象是该接口的实现，只要结构符合即可
  - 可选`?`, `readonly properties`
  ```js
  // const to variable, readonly to property
  interface SquareConfig {
    readonly color?: string;
    width?: number;
  }
  ```
  - 多余属性的检查
    多余属性会报错，可以通过断言绕过
    ```js
    // opacity属性是SquareConfig接口没有的，但是这么断言以后可以绕过createSquare函数多余参数的检查
    let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
    ```
  - 未知属性
  ```js
  interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
  }
  ```
  - 函数类型
  ```js
  // 定义参数类型和返回值类型
  interface SearchFunc {
    (source: string, subString: string): boolean;
  }
  // 瞬间回到初学c的时候
  int main(int a, int b) {
    return a + b
  }
  ```
  - 可索引类型
  ```js
  interface St{
    [x: number]: string,
    [x: string]: number,
    lslslsl: string // 报错，因为上一条类型冲突
  }
  ```
  - class类型
  ```js
  // in a word 这个类实现了这个接口
  interfaceB{
    variable: number,
    func(i: number)
  }
  class A implements interfaceB{

  }
  ```
    - Interfaces describe the public side of the class, rather than both the public and private side
    - A class has two types: the type of the static side and the type of the instance side
    - When a class implements an interface, only the instance side of the class is checked

  - extend
    - 可以extend多个接口，但是如果这多个接口有相同的属性名时会报错
    ```js
    interface Shape {
        color: string;
    }

    interface PenStroke {
        penWidth: number;
    }

    interface Square extends Shape, PenStroke {
        sideLength: number;
    }
    let square = <Square>{};
    ```
    - interface extending classes
      接口extend类时相当于在不实例化类的前提下声明了类的所有成员（包括私有和受保护的），这就导致只有这个类以及他的子类可以去实现这个接口
      ```js
      class Control {
          private state: any;
      }

      interface SelectableControl extends Control {
          select(): void;
      }

      class Button extends Control implements SelectableControl {
          select() { }
      }

      // Error: Property 'state' is missing in type 'Image'.
      class Image implements SelectableControl {
          select() { }
      }
      ```

- class
  ```js
  var Greeter = /** @class */ (function () {
    function Greeter() {
    }
    Greeter.prototype.greet = function () {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    };
    Greeter.standardGreeting = "Hello, there";
    return Greeter;
  }());
  ```
  上为class的es5实现
  class返回的是构造函数/静态属性和实例属性两部分
  ```ts
  let greeterMaker: typeof Greeter = Greeter; // var greeterMaker = Greeter; 构造函数及静态属性
  let ge = new Greeter; // 实例化，原型上的属性，基于原型的继承
  ```

- generic
  - generic class
  generic class's static members can not use the class's type parameter
    ```ts
    class GenericNumber<T> {
      zeroValue: T;
      add: (x: T, y: T) => T;
    }
    ```
  - generic constrained
    // T extends interface, 相当于为T设置类型
    interface Lengthwise {
      length: number;
    }

    function loggingIdentity<T extends Lengthwise>(arg: T): T {
        console.log(arg.length);  // Now we know it has a .length property, so no more error
        return arg;
    }
    ```
  - confused
  ```ts
  function create<T>(c: {new(): T; }): T {
    return new c();
  }
  ```

- type compatiblity
  ```ts
  interface Named {
    name: string;
  }
  let x: Named;
  // y's inferred type is { name: string; location: string; }
  let y = { name: "Alice", location: "Seattle" };
  x = y;
  function greet(n: Named) {
    alert("Hello, " + n.name);
  }
  greet(y); // OK
  ```
  只check Named类型(target type)，所以这个时候不会因为参数报错


### node

#### node project princeples

- organize your files around features, not roles
- don't put logic in `index.js` file
- put your test files next to the implementation
- use a `config` directory
- put your long npm script in a `scripts` directory

#### koa

- assumption
  - node: 8.8.1
  - koa: 2.4.1

-
