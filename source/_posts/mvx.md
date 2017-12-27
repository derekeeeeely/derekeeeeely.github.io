---
title: MV*浅析
tags:
  - mv*
  - backbone
  - vue
  - react
categories:
  - code
  - note
abbrlink: 98f5a61f
date: 2017-12-18 16:17:17
---

&emsp;&emsp;在git上看到一个[to do mvc](https://github.com/tastejs/todomvc)的repo，一时有了兴致，记录下一些点
<!-- more -->

### 历史

#### 静态/展示->动态/交互

- web1.0时代，服务端直接将数据填充进模板，生成Html返回给浏览器，静态网页，纯展示
- web2.0时代，动态网页，前端向服务端请求数据，然后自己处理数据，提供了用户和浏览器的交互

#### 前端mv*的必要性

- 前端进入到需要保存数据、处理数据和生成视图的web2.0时代
- webapp的概念，web不再是页面级，而是application级别，所以需要思考应用的管理，数据的流向等

![](http://opo02jcsr.bkt.clouddn.com/ff5d456ff2abab2a42527f3bae237132.png)

### todo mv*

  - 其实差别就前端如何管理数据，以及数据如何和视图联系起来


#### backbone

  - 只有Model和View
  - 结构
    ├── index.html
    ├── js
        ├── app.js
        ├── collections
        │   └── todos.js
        ├── models
        │   └── todo.js
        ├── routers
        │   └── router.js
        └── views
            ├── app-view.js
            └── todo-view.js

  - 以新增为例
    ```js
    // views/app-view.js
    // 绑定节点
    el: '.todoapp',
    // 声明事件
    events: {
      'keypress .new-todo': 'createOnEnter',
    }
    // 用户在view层enter，调用collection的方法创建新的todo model
    createOnEnter: function (e) {
      if (e.which === ENTER_KEY && this.$input.val().trim()) {
        app.todos.create(this.newAttributes());
        this.$input.val('');
      }
    }

    // collections/todos.js
    var Todos = Backbone.Collection.extend({
      // Reference to this collection's model.
      model: app.Todo,
    })

    // models/todo.js
    app.Todo = Backbone.Model.extend({
      defaults: {
        title: '',
        completed: false
      }
    })

    // viwes/todo-views.js
    // 绑定模板
    template: _.template($('#item-template').html()),
    // 监听model的change，注册回调render
    initialize: function () {
      this.listenTo(this.model, 'change', this.render);
    }
    // render方法，将model数据填充
    render: function () {
      if (this.model.changed.id !== undefined) {
        return;
      }
      this.$el.html(this.template(this.model.toJSON()));
      return this;
    }
    ```

  - 定义model和collection，在view层通过事件触发model层行为，改变数据后，通过事先注册的监听model变化的回调函数来更新view

#### vue

  - model, view, modelview (mvvm)
  - 结构
    ├── index.html
    ├── js
    │   ├── app.js
    │   ├── routes.js
    │   └── store.js
  - 新增为例
  ```js
  // app.js
  // vue对象，连接view和model的vm
  exports.app = new Vue({
    // 绑定节点
    el: '.todoapp',
    // 定义数据模型
    data: {
      todos: todoStorage.fetch(),
      newTodo: '',
    },
    // 修改数据模型的方法
    methods: {
      addTodo: function() {
        var value = this.newTodo && this.newTodo.trim();
        if (!value) {
          return;
        }
        this.todos.push({ id: this.todos.length + 1, title: value, completed: false });
        this.newTodo = '';
      }
    }
  })

  // index.html
  // v-modal将数据（vm）和节点（v）绑定，通过事件触发vue对象的方法改变数据，并通过观察者模式和自定义的访问器实现view的更新
  <input class="new-todo" autofocus autocomplete="off" v-model="newTodo" @keyup.enter="addTodo">
  ```
  - vm实际上是简化的controller，将model数据简单处理，为view提供正确的数据

#### react

  - react只是视图层解决方案，本质是将图形界面（GUI）函数化。
  - view是state的输出，引自阮一峰老师 ``js view = f(state)``
  - 组件state/props用于数据存储流转，render函数输出view，事件驱动产生数据变化，setState重新render...
  - 结构
  ├── index.html
  ├── js
  │   ├── app.jsx
  │   ├── footer.jsx
  │   ├── todoItem.jsx
  │   ├── todoModel.js
  │   └── utils.js
  ```js
  // todoModel.js
  // 定义model，在原型上定义一些方法，比如存数据到localstorage里持久化
  app.TodoModel = function (key) {
    this.key = key;
    this.todos = Utils.store(key);
    this.onChanges = [];
  };
  // 注册回调进行订阅
  app.TodoModel.prototype.subscribe = function (onChange) {
    this.onChanges.push(onChange);
  };
  // 调用inform时通知各个订阅者，执行回调
  app.TodoModel.prototype.inform = function () {
    Utils.store(this.key, this.todos);
    this.onChanges.forEach(function (cb) { cb(); });
  };
  // app.jsx
  var TodoApp = React.createClass({
    // 初始化state
    getInitialState: function () {
      return {
        newTodo: ''
      };
    },
    // 事件触发，setState，重新render
    handleChange: function (e) {
      this.setState({newTodo: e.target.value});
    },
    render: function () {
      return <div>
        <input onChange={this.handleChange} />
        <div>{this.state.newTodo}</div>
      </div>
    }
  })
  var model = new app.TodoModel('react-todos');
  function render() {
    React.render(
      <TodoApp model={model}/>,
      document.getElementsByClassName('todoapp')[0]
    );
  }
  model.subscribe(render);
  render()
  ```

#### flux

  ![](http://opo02jcsr.bkt.clouddn.com/a1f71617fc991252b13bbb92ecf1f910.png)
  - 不同组件的state放在一个外部的通用的store里
  - 每个组件订阅该store的一部分
  - 组件内通过dispatch一个action去触发store的更新

#### redux

  ![](http://opo02jcsr.bkt.clouddn.com/fd0c3c8b8fdf803f8cc2adc1698a799e.png)

  - 状态都存放在store中，组件的重新渲染都由状态改变来触发
  - 用户通过在view层dispatch action触发reducer，在reducer中 new = f(old) 计算得到新的state

  ![](http://opo02jcsr.bkt.clouddn.com/98fc05f593a3c073b475091c9849ac92.png)

  - UI组件和容器组件
    - UI组件负责页面外观
    - 容器组件负责数据和行为，订阅store，处理store的数据然后传递给UI组件，例如connect方法
  - Provider和connect
    - 前者将react应用包一层，使得组件和store可被连接，同时传入store
    ```js
    const store = createStore(reducer);
    ReactDOM.render(
      <Provider store={store}>
        <App />
      </Provider>,
      document.body.appendChild(document.createElement('div'))
    );
    ```
    - store由redux得createStore方法提供，该方法接收reducer作为参数，实际上相当于定义了action list对应的处理对象和方案
    - connect将UI组件和容器组件连接，使得在我们的UI/业务组件中能够获取到最初定义好的store，并在UI组件中通过dispatch action去对上一条所说的处理对象执行处理方案
  - reducer
    -纯函数
    ```js
    // 接收旧的state和action，返回新的state
    function reducer(state = {
      text: '你好，访问者',
      name: '访问者'
    }, action) {
      switch (action.type) {
        case 'change':
          return {
            name: action.payload,
            text: '你好，' + action.payload
          };
      }
    }
    ```

    - 典型的MV*，model对应于store，view对应于业务组件，连接起来或者说数据处理交由action reducer完成，适合于大型应用。

#### Mobx

  - UI 层是观察者，Store 是被观察者。
  - 🌰 下回分解
  ```js
  const {observable} = mobx;
  const {observer} = mobxReact;
  const person = observable({name: "张三", age: 31});
  const App = observer(
    ({ person }) => <h1>{ person.name }</h1>
  );
  ReactDOM.render(<App person={person} />, document.body);
  person.name = "李四";
  ```


