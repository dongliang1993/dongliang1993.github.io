---
title: redux 源码分析 (一)
date: 2017-12-13 10:06:50
tags:
---
首先我们看一下 redux 的代码结构:
![](//dongliang1993.github.io/img/redux-源码/redux-01.png)

├── applyMiddleware.js  // API: applyMiddleware 中间件，对store.dispatch方法进行了改造。作用是将所有中间件组成一个数组，依次执行
├── bindActionCreators.js  // API: bindActionCreators 把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，而且不希望把 Redux store 或 dispatch 传给它
├── combineReducers.js  // API: combineReducers 把一个由多个不同 reducer 函数作为 value 的 object，合并成一个 reducer 函数
├── compose.js     // API: compose  从右到左来组合多个函数。
├── createStore.js    // API: createStore 创建一个 Redux store 来以存放应用中所有的 state。
├── index.js     // 入口文件，负责集中暴露 API，例如 createStore 、combineReducers 等
└── utils  // 工具包
    ├── actionTypes.js  // Redux 保留的私有 action 类型。
    ├── isPlainObject.js // 判断是否为普通对象
    └── warning.js  // 在控制台中打印警告

<!-- more -->

## index.js 文件

这个入口文件只有两个功能：
1. 暴露所有的 API
2. 对代码是否压缩进行判断

### 第一个功能对应的代码：
```js
// index.js
// 入口文件
// 首先引入相应的模块，具体模块的内容后续会详细分析
import createStore from './createStore'
import combineReducers from './combineReducers'
import bindActionCreators from './bindActionCreators'
import applyMiddleware from './applyMiddleware'
import compose from './compose'
import warning from './utils/warning'
import __DO_NOT_USE__ActionTypes from './utils/actionTypes'

...

// 导出相应的功能
export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

就可以看到，最后我们导出了一个对象，这也是我们为什么可以用如下语法来引入对应方法的原因：
```js
import { createStore, combineReducers } from 'redux
```

本质就是从导出的这个对象上解构出想要的方法

可能有的小伙伴对 export、export default 等这些语法不是很了解，后面我会整理一下相关的教程

### 第二个功能对应的代码：
```js
/*
 * This is a dummy function to check if the function name has been altered by minification.
 * If the function has been minified and NODE_ENV !== 'production', warn the user.
 */
// 这是一个虚拟函数来检查函数名是否被压缩了
// 如果当前不是 production 环境而代码却被压缩了，警告 user
function isCrushed() {}
// 主要是通过 函数名是否还是原名来判断
if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning(
    "You are currently using minified code outside of NODE_ENV === 'production'. " +
      'This means that you are running a slower development build of Redux. ' +
      'You can use loose-envify (https://github.com/zertosh/loose-envify) for browserify ' +
      'or DefinePlugin for webpack (http://stackoverflow.com/questions/30030031) ' +
      'to ensure you have the correct code for your production build.'
  )
}
```

我尝试着来分析一下这段代码的意义：

我们都知道，在项目上线前（生产环境），我们为了安全，都会进行代码压缩、加密、混淆，比如下面这段代码压缩前后可能会是这个样子：
```js
function echo(stringA, stringB) {
	const hello = "你好"
	console.log(hello)
}
// 压缩后
function a(b, c) {
	const d = "你好"
	console.log(d)
}
```

但是，如果是开发、测试环境，其实是没有必要的。这段代码就是用来检测在非生产环境下代码是否被压缩，主要目的是担心你在开发环境用了 redux 的压缩版本，导致在你压缩项目时 redux 再次被压缩出现报错

function 有 name 属性，还有 arguments 、length 等属性 ，参考https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/name

这里就是通过 函数名是否还是原名来判断

## createStore.js 文件

我们先大体看一下这个这个文件的输入和输出：
```js
export default function createStore(reducer, preloadedState, enhancer) {
  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```

整个 createStore.js 文件最后就暴露出来 createStore 这一个函数

createStore 函数接受三个参数：reducer，preloadedState，enhancer。分别对应全局 reducer，store 中初始的 state 和 applyMiddleware 中间件函数。三个参数中，只有第一个参数是必传的，后面两个参数可选。函数的返回值是一个对象，一共暴露出来了五个方法，但是我们常用的一般只有前三个

我们使用 createStore 创建 Store 时，一般有三种用法
```js
// 第一种用法
// 只有一个reducer
const store = createStore(
  reducer,
)

// 第二种用法
// 传入一个 initState
const store = createStore(
  reducer,
  initState,
)
// 传入一个 enhancer
const store = createStore(
  reducer,
  applyMiddleware(...middleware),
)

// 第三种用法
const store = createStore(
  reducer,
  initState,
  applyMiddleware(...middleware),
)
```

Store 保存了应用所有 state 的对象。改变 state 的惟一方法是 dispatch action。你也可以 subscribe 监听 state 的变化，然后更新 UI。

Store 有以下职责，分别对应 Store 身上暴露出来的几个函数：
1. 维持应用的 state；
2. 提供 getState() 方法获取 state；
3. 提供 dispatch(action) 方法更新 state；
4. 通过 subscribe(listener) 注册监听器;
5. 通过 subscribe(listener) 返回的函数注销监听器。

大致的功能说完了，接下来让我们看一下源码是怎么实现的：

```js
  // 调整参数，验证参数类型
  // 处理的第二种用法中的情况二： 传入一个 reducer 和 一个 enhancer
  // 没有传入预先加载的state
  // 并且第二个参数是一个函数的话，则把第二个参数为功能增强函数enhancer
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
  // 如果有 enhancer 的话，要对 enhancer 进行处理
  if (typeof enhancer !== 'undefined') {
    // 判断 enhancer 必须是一个函数
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    // 这是一个很重要的处理，它将 createStore 方法作为参数传入enhancer函数，
    // 并且执行enhancer
    // 这里主要是提供给redux中间件的使用，以此来达到增强整个redux流程的效果
    // 通过这个函数，也给redux提供了无限多的可能性
    return enhancer(createStore)(reducer, preloadedState)
  }
  // reducer 必须是一个函数，否则报错
  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }
  
```

我们后面分析道 enhancer 源码的时候，会对这部分再详细的讲解

接着往下看
```js
// 将传入的reducer缓存到currentReducer变量中
let currentReducer = reducer
// 将传入的preloadedState缓存到currentState变量中
let currentState = preloadedState
// 定义当前的监听者队列
let currentListeners = []
// 定义下一个循环的监听者队列
let nextListeners = currentListeners
// 定义一个判断是否在dispatch的标志位
let isDispatching = false
// 判断是否能执行下一次监听队列
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    // 这里是将当前监听队列通过拷贝的形式赋值给下次监听队列，这样做是为了防止在当前队列执行的时候会影响到自身，所以拷贝了一份副本
    nextListeners = currentListeners.slice()
  }
}

...

// When a store is created, an "INIT" action is dispatched so that every
// reducer returns their initial state. This effectively populates
// the initial state tree.
// 当 store 被创建的时候， 会派发一个 "INIT" 的 action，
// 所以每个 reducer 都被执行，然后 return 出他们的初始值，
// 这些初始值组成了初始的 state tree
// dispatch一个初始化的action
dispatch({ type: ActionTypes.INIT })
```

这里可以看到，在我们创建 store 的时候，就已经执行了第一次的dispatch ，
派发了一个 "INIT" action 。然后回执行 reducer（如果是 combineReducers 合成的，也会执行）。这一步的作用是用来初始化 state 树。这也是为什么我们即使没有传入 initState 也可以拥有初始化 state 的原因

接下来我们理所应当的进入到 dispatch 函数来一探究竟
```js
 /**
   * Dispatches an action. It is the only way to trigger a state change.
   * The `reducer` function, used to create the store, will be called with the
   * current state tree and the given `action`. Its return value will
   * be considered the **next** state of the tree, and the change listeners
   * will be notified.
   * 
   * 派发一个 action ，这是触发 state 改变的唯一方式
   * 被用来创建一个 store 的 reducer 函数，会被一个 action 来触发调用
   * 这个 reducer 的返回值就是下一次 state 的值，并且 listeners 也会被通知调用
   * 
   * The base implementation only supports plain object actions. If you want to
   * dispatch a Promise, an Observable, a thunk, or something else, you need to
   * wrap your store creating function into the corresponding middleware. For
   * example, see the documentation for the `redux-thunk` package. Even the
   * middleware will eventually dispatch plain object actions using this method.
   *
   * 这个最基本的实现只支持扁平化的 actions，如果你想派发一个 promise、Observable、thunk
   * 或者其他东西，你需要用中间件包装你的 createStore 函数
   * 即使被中间件包装，最后还是派发了一个扁平的 action
   * 
   * @param {Object} action A plain object representing “what changed”. It is
   * a good idea to keep actions serializable so you can record and replay user
   * sessions, or use the time travelling `redux-devtools`. An action must have
   * a `type` property which may not be `undefined`. It is a good idea to use
   * string constants for action types.
   *
   * @returns {Object} For convenience, the same action object you dispatched.
   *
   * Note that, if you use a custom middleware, it may wrap `dispatch()` to
   * return something else (for example, a Promise you can await).
   */
  // redux 中通过 dispatch 一个 action ，来触发对 store 中的 state 的修改
  // 参数就是一个 action
  function dispatch(action) {
    // 这里判断一下 action 是否是一个扁平的对象，如果不是则抛出错误
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
      )
    }
    // action中必须要有type属性，否则抛出错误
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }
    // 如果上一次dispatch还没结束，则不能继续dispatch下一次
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      // 将isDispatching设置为true，表示当次dispatch开始
      isDispatching = true
      // 利用传入的reducer函数处理state和action，返回新的state
      // 推荐不直接修改原有的currentState
      currentState = currentReducer(currentState, action)
    } finally {
      // 当次的dispatch结束
      isDispatching = false
    }
    // 每次dispatch结束之后，就执行监听队列中的监听函数
    // 将nextListeners赋值给currentListeners，保证下一次执行ensureCanMutateNextListeners方法的时候会重新拷贝一个新的副本
    // 简单粗暴的使用for循环执行
    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }
    // 最后返回action
    return action
  }
```

然后是 getState 函数
```js
/**
 * Reads the state tree managed by the store.
 *
 * @returns {any} The current state tree of your application.
 */
// 获取当前的state
function getState() {
  if (isDispatching) {
    throw new Error(
      'You may not call store.getState() while the reducer is executing. ' +
        'The reducer has already received the state as an argument. ' +
        'Pass it down from the top reducer instead of reading it from the store.'
    )
  }

  return currentState
}

```

subscribe 函数
```js
/**
 * Adds a change listener. It will be called any time an action is dispatched,
 * and some part of the state tree may potentially have changed. You may then
 * call `getState()` to read the current state tree inside the callback.
 *
 * 添加一个事件监听。每次派发一个 action 或者 state 树发生变化都会依次执行
 * listener 里面的所有监听函数
 * 
 * You may call `dispatch()` from a change listener, with the following
 * caveats:
 *
 * 1. The subscriptions are snapshotted just before every `dispatch()` call.
 * If you subscribe or unsubscribe while the listeners are being invoked, this
 * will not have any effect on the `dispatch()` that is currently in progress.
 * However, the next `dispatch()` call, whether nested or not, will use a more
 * recent snapshot of the subscription list.
 *
 * 2. The listener should not expect to see all state changes, as the state
 * might have been updated multiple times during a nested `dispatch()` before
 * the listener is called. It is, however, guaranteed that all subscribers
 * registered before the `dispatch()` started will be called with the latest
 * state by the time it exits.
 *
 * @param {Function} listener A callback to be invoked on every dispatch.
 * @returns {Function} A function to remove this change listener.
 */
// 往监听队列里面去添加监听者
function subscribe(listener) {
  // 监听者必须是一个函数
  if (typeof listener !== 'function') {
    throw new Error('Expected listener to be a function.')
  }
  // 如果正在 isDispatching
  if (isDispatching) {
    throw new Error(
      'You may not call store.subscribe() while the reducer is executing. ' +
        'If you would like to be notified after the store has been updated, subscribe from a ' +
        'component and invoke store.getState() in the callback to access the latest state. ' +
        'See http://redux.js.org/docs/api/Store.html#subscribe for more details.'
    )
  }
  // 声明一个变量来标记是否已经subscribed，通过闭包的形式被缓存
  let isSubscribed = true
  // 创建一个当前currentListeners的副本，赋值给nextListeners
  ensureCanMutateNextListeners()
  // 将监听者函数push到nextListeners中
  nextListeners.push(listener)
  // 返回一个取消监听的函数
  // 原理很简单就是从将当前函数从数组中删除，使用的是数组的splice方法
  return function unsubscribe() {
    if (!isSubscribed) {
      return
    }

    if (isDispatching) {
      throw new Error(
        'You may not unsubscribe from a store listener while the reducer is executing. ' +
          'See http://redux.js.org/docs/api/Store.html#subscribe for more details.'
      )
    }

    isSubscribed = false

    ensureCanMutateNextListeners()
    const index = nextListeners.indexOf(listener)
    nextListeners.splice(index, 1)
  }
}
```

```js
  /**
   * Replaces the reducer currently used by the store to calculate the state.
   *
   * You might need this if your app implements code splitting and you want to
   * load some of the reducers dynamically. You might also need this if you
   * implement a hot reloading mechanism for Redux.
   *
   * @param {Function} nextReducer The reducer for the store to use instead.
   * @returns {void}
   */
  // replaceReducer方法，顾名思义就是替换当前的reducer处理函数
  function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.REPLACE })
  }
```


## combineReducers.js

首先看一下整体
```js
...
export default function combineReducers(reducers) {
  ...
}
```

和之前一样，这个文件只导出了单独的 combineReducers 函数

combineReducers 只接受一个对象作为参数, 我们通常这么使用 
```js
const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

这两种写法是一样的

```js

/**
 * Turns an object whose values are different reducer functions, into a single
 * reducer function. It will call every child reducer, and gather their results
 * into a single state object, whose keys correspond to the keys of the passed
 * reducer functions.
 *
 * 把一个对象转换成单个的 reducer 函数，这个对象的 value 是拆分出来的单个reducer。combineReducers 函数会调用所有的 reducer，然后把返回值收集起来
 * 组成一个单个的 state object，key 值与 参数中 reducer functions 的key
 * 值一一对应
 * 
 * @param {Object} reducers An object whose values correspond to different
 * reducer functions that need to be combined into one. One handy way to obtain
 * it is to use ES6 `import * as reducers` syntax. The reducers may never return
 * undefined for any action. Instead, they should return their initial state
 * if the state passed to them was undefined, and the current state for any
 * unrecognized action.
 *
 * @returns {Function} A reducer function that invokes every reducer inside the
 * passed object, and builds a state object with the same shape.
 */
// 导出combineReducers方法，接受一个参数reducers对象
export default function combineReducers(reducers) {
  // 获取reducers对象的key值
  const reducerKeys = Object.keys(reducers)
  // 定义一个最终要返回的 reducers 对象
  const finalReducers = {}
  // 遍历这个reducers对象的key
  for (let i = 0; i < reducerKeys.length; i++) {
    // 缓存每个key值
    const key = reducerKeys[i]
    // 如果不是生产环境
    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }
    // 相应key的值是个函数，则将改函数缓存到finalReducers中
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  // 上面那一步其实主要是起到过滤作用
  // 把 reducers 中 key-val 对中 value 不是函数的过滤掉了

  // 获取finalReducers的所有的key值，缓存到变量finalReducerKeys中
  const finalReducerKeys = Object.keys(finalReducers)

  let unexpectedKeyCache
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  // 定义一个变量，用于缓存错误对象
  let shapeAssertionError
  try {
    // 做错误处理，详情看后面assertReducerShape方法
    // 主要就是检测，
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  // combineReducers 调用后还是会返回一个函数
  // 这样就和 createStore 中的参数对应上了
  return function combination(state = {}, action) {
    // 如果有错误，则抛出错误
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    if (process.env.NODE_ENV !== 'production') {
      // 获取警告提示
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    // 定义一个变量来表示state是否已经被改变
    let hasChanged = false
    // 定义一个变量，来缓存改变后的state
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      // 获取有效的reducer的key值
      const key = finalReducerKeys[i]
      // 根据key值获取对应的reducer函数
      const reducer = finalReducers[key]
      // 根据 key 值获取每个 reducer 对应的 state 的部分
      const previousStateForKey = state[key]
      // 执行 reducer 函数，获取相应模块的state
      const nextStateForKey = reducer(previousStateForKey, action)
      // 如果获取的state是undefined,则抛出错误
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      // 将获取到的新的state赋值给新的state对应的模块，
      // key则为当前reducer的key
      nextState[key] = nextStateForKey
      // 判读state是否发生改变
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    // 如果state发生改变则返回新的state，否则返回原来的state
    return hasChanged ? nextState : state
  }
}

```


## bindActionCreators.js

首先先认识 actionCreators, 简单来说就是创建 action 的方法，redux 的 action 是一个对象，而我们经常使用一些函数来创建这些对象，则这些函数就是 actionCreators

而这个文件实现的功能，是根据绑定的 actionCreator，来实现自动dispatch的功能

actionCreators 接受两个参数，第一个参数是个对象，第二个参数是 dispatch
```js
// actionCreators
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  };
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  };
}
```

第一个参数的格式
```js
{
  addTodo: function addTodo(text) {
    return {
      type: 'ADD_TODO',
      text
    };
  },
  removeTodo: function removeTodo(id) {
    return {
      type: 'REMOVE_TODO',
      id
    };
  }
}
```

用法：
boundActionCreators..addTodo('Use Redux')

```js
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}
/**
 * Turns an object whose values are action creators, into an object with the
 * same keys, but with every function wrapped into a `dispatch` call so they
 * may be invoked directly. This is just a convenience method, as you can call
 * `store.dispatch(MyActionCreators.doSomething())` yourself just fine.
 *
 * For convenience, you can also pass a single function as the first argument,
 * and get a function in return.
 *
 * @param {Function|Object} actionCreators An object whose values are action
 * creator functions. One handy way to obtain it is to use ES6 `import * as`
 * syntax. You may also pass a single function.
 *
 * @param {Function} dispatch The `dispatch` function available on your Redux
 * store.
 *
 * @returns {Function|Object} The object mimicking the original object, but with
 * every action creator wrapped into the `dispatch` call. If you passed a
 * function as `actionCreators`, the return value will also be a single
 * function.
 */
// 对外暴露这个bindActionCreators方法
export default function bindActionCreators(actionCreators, dispatch) {
  // 如果传入的actionCreators参数是个函数，则直接调用bindActionCreator方法
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }
  // 错误处理
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }
  // 如果actionCreators是一个对象，则获取对象中的key
  const keys = Object.keys(actionCreators)
  // 定义一个缓存对象
  const boundActionCreators = {}
  // 遍历actionCreators的每个key
  for (let i = 0; i < keys.length; i++) {
    // 获取每个key
    const key = keys[i]
    // 根据每个key获取特定的actionCreator方法
    const actionCreator = actionCreators[key]
    // 如果actionCreator是一个函数，则直接调用bindActionCreator方法，将返回的匿名函数缓存到boundActionCreators对象中
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  // 最后返回boundActionCreators对象
  // 用户获取到这个对象后，可拿出对象中的每个key的对应的值，也就是各个匿名函数，执行匿名函数就可以实现dispatch功能
  return boundActionCreators
}

```


## compose

```js
compose(
  applyMiddleware(thunk),
  DevTools.instrument()
)
```

```js
/**
 * Composes single-argument functions from right to left. The rightmost
 * function can take multiple arguments as it provides the signature for
 * the resulting composite function.
 *
 * 合成多个只接受一个参数的函数，它的返回值将作为一个参数提供给它左边的函数，
 * 以此类推。例外是最右边的参数可以接受多个参数，因为它将为由此产生的函数提供签名。
 * 
 * （译者注：compose(funcA, funcB, funcC) 形象为 compose(funcA(funcB(funcC())))）
 * 
 * @param {...Function} funcs The functions to compose.
 * @returns {Function} A function obtained by composing the argument functions
 * from right to left. For example, compose(f, g, h) is identical to doing
 * (...args) => f(g(h(...args))).
 */

export default function compose(...funcs) {
  // 判断函数数组是否为空
  if (funcs.length === 0) {
    return arg => arg
  }
  // 如果函数数组只有一个元素，则直接执行
  if (funcs.length === 1) {
    return funcs[0]
  }
  // 否则，就利用reduce方法执行每个中间件函数，并将上一个函数的返回作为下一个函数的参数
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```