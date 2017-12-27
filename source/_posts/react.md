---
title: React小笔记
tags:
  - react
  - source code
categories:
  - code
  - note
abbrlink: 19656fd5
date: 2017-12-26 13:22:43
---

&emsp;&emsp;这是一篇错漏百出，凌乱不堪的react源码学习笔记，推荐这个git库[read-react-source-code](https://github.com/numbbbbb/read-react-source-code/blob/master/02-how-render-works.md)，值得一看
<!-- more -->

### brief description

  ```js
    ReactDOM.render(
      <h1>Hello world!</h1>,
      document.getElementById('example')
    );
  ```

- get started: element

  create element
  > question: why validator before call render ?


  ```js
  ReactElementValidator = {
    createElement: function (type, props, children) {
      // @arguments ['h1', null, 'hello world!']
      ReactElement.createElement.apply(this, arguments)
      'validate keys'
      'validate proptypes'
      return element
    }
  }

  ReactElement.createElement = function (type, config, children) {
    'merge children into props'
    'add ref and key to props'
    return ReactElement(type, key, ref, self, source, ReactCurrentOwner.current, props)
  }

  ReactElement = function(type, key, ref, self, source, owner, props) {
    'add some property'
    'freeze element and element props'
    return element
  }
  ```
  see element's look

  ![](http://opo02jcsr.bkt.clouddn.com/1e476f98b0e1db760c4eb99c75a6bc9c.png)

  > question: how we jump to render ? debug shows ReactPerf.measure called it, but how we get into measure ?


- render: instance
  ```js
  render: function (nextElement, container, callback) {
    return ReactMount._renderSubtreeIntoContainer(null, nextElement, container, callback);
  }
  ```

  ```js
  _renderSubtreeIntoContainer: function (parentComponent, nextElement, container, callback) {
    'some validation'
    // nextElement here is in the picture above, while nextWrappedElement is below
    // TopLevelWrapper: function(){  this.rootID = topLevelRootCounter++ }
    var nextWrappedElement = ReactElement(TopLevelWrapper, null, null, null, null, null, nextElement);
    ...cause we get only one node here, we'll mention these details later
  }
  ```
  > question: why wrap ?
    to distinguish between ReactCompositeComponent/ReactNativeComponent/ReactDOMComponent ?


  ![](http://opo02jcsr.bkt.clouddn.com/c2196a14908262f98d90f91789fa066c.png)

  ```js
    var component = ReactMount._renderNewRootComponent(nextWrappedElement, container, shouldReuseMarkup, nextContext)._renderedComponent.getPublicInstance();

    _renderNewRootComponent: function (nextElement, container, shouldReuseMarkup, context) {
      'check container node type'

      // Listens to window scroll and resize events. We cache scroll values so that application code can access them without triggering reflows.
      ReactBrowserEventEmitter.ensureScrollValueMonitoring();

      // Given a ReactNode, create an instance that will actually be mounted.
      var componentInstance = instantiateReactComponent(nextElement);

      // The initial render is synchronous but any updates that happen during rendering, in componentWillMount or componentDidMount, will be batched according to the current batching strategy.
      ReactUpdates.batchedUpdates(batchedMountComponentIntoNode, componentInstance, container, shouldReuseMarkup, context);

      return componentInstance
    }

    // ReactCompositeComponentWrapper for example
    var ReactCompositeComponentWrapper = function (element) {
      this.construct(element);
    };

  ```

  instance may be this

  ![](http://opo02jcsr.bkt.clouddn.com/aaf009ed7c2590e24c412210a0f614b7.png)


  ```js
    // this is abc about instantiateReactComponent which is important enough to own an unique block
    function instantiateReactComponent(node, shouldHaveDebugID) {
      if (node === null || node === false) {
        instance = ReactEmptyComponent.create(instantiateReactComponent);
      } else if (typeof node === 'object') {
        var element = node;
        // Special case string values
        if (typeof element.type === 'string') {
          instance = ReactNativeComponent.createInternalComponent(element);
        } else if (isInternalComponentType(element.type)) {
          instance = new element.type(element);
        } else {
          instance = new ReactCompositeComponentWrapper(element);
        }
      } else if (typeof node === 'string' || typeof node === 'number') {
        instance = ReactNativeComponent.createInstanceForText(node);
      }
    }
  ```

- mount: component into dom

  ```js
    function batchedUpdates(callback, a, b, c, d, e) {
      ensureInjected();
      batchingStrategy.batchedUpdates(callback, a, b, c, d, e);
    }

    ReactDefaultBatchingStrategy = {
      batchedUpdates: function (callback, a, b, c, d, e) {
        'check sth'
        transaction.perform(callback, null, a, b, c, d, e);
      }
    }
    // perform call callback batchedMountComponentIntoNode
    batchedMountComponentIntoNode() {
      var transaction = ReactUpdates.ReactReconcileTransaction.getPooled(
        /* useCreateElement */
        !shouldReuseMarkup && ReactDOMFeatureFlags.useCreateElement,
      );
      transaction.perform(mountComponentIntoNode, null, componentInstance, container, transaction, shouldReuseMarkup, context);
    }
  ```

  here is the transaction:

  ![](http://opo02jcsr.bkt.clouddn.com/4ce81da62d2ed9d2c10f2c9531f236dc.png)

  ```js
    // before this part, react has done quite many things, but we are not going to talk about it here
    // Mounts this component and inserts it into the DOM.
    function mountComponentIntoNode(wrapperInstance, container, transaction, shouldReuseMarkup, context) {
      'sth'
      var markup = ReactReconciler.mountComponent(wrapperInstance, transaction, null, ReactDOMContainerInfo(wrapperInstance, container), context, 0 /* parentDebugID */
    }

    function ReactDOMContainerInfo(topLevelWrapper, node) {
      'validateDOMNesting'
      return info
    }

    // Initializes the component, renders markup, and registers event listeners.
    mountComponent: function (internalInstance, transaction, nativeParent, nativeContainerInfo, context) {
      var markup = internalInstance.mountComponent(transaction, nativeParent, nativeContainerInfo, context);
    }

    // Initializes the component, renders markup, and registers event listeners.
    internalInstance.mountComponent: function (transaction, hostParent, hostContainerInfo, context) {
      'not patient enough to follow these code, yet...'
      return markup
    }

  ```

  ReactDOMContainerInfo is here:

  ![](http://opo02jcsr.bkt.clouddn.com/36913673cf38732f647db1065385f898.png)

  markup is here:

  ![](http://opo02jcsr.bkt.clouddn.com/6bac0665c898e51e6144cef8a50a351f.png)

- somebody says

  ![](http://opo02jcsr.bkt.clouddn.com/0069274d07a5cb77ad10c284304c7896.jpg)

  my call stack

  ![](http://opo02jcsr.bkt.clouddn.com/32b2d1f0d28e6521735665a2cf90bb18.png)

- nested

  ```js
    class Demo extends React.Component {
      constructor(props) {
        super(props);
        this.state = {name: 'John'};
      }

      render() {
        return (
          <h1>
            Hello, {this.state.name}!
          </h1>
        )
      }

      componentDidMount() {
        this.setState({
          name: 'Jack'
        })
      }
    }

    ReactDOM.render(
      <Demo />,
      document.getElementById('example')
    );
  ```
  render的时候demo作为type存入elementTree
  element => componentInstance的时候调batchupdate，开启事务，do sth like add listener...
  回调batchedMountComponentIntoNode，开启事务
  ReactReconciler.mountComponent => ReactCompositeComponentMixin.mountComponent, process props/context
  performInitialMount => _processChildContext 处理child
  Demo.render => 重复以上
  mountImageIntoNode => dom => done

  ![](http://opo02jcsr.bkt.clouddn.com/4dfe91e0bfd1db022b2d569bb56fb975.png)



- 综上，简直乱七八糟，不清不楚...还是看看别人总结的...
  参考 https://github.com/numbbbbb/read-react-source-code/blob/master/02-how-render-works.md
