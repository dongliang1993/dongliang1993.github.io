---
title: promise 笔记
date: 2018-03-06 16:06:37
tags:
---

* Promise 是一个构造函数，接收一个函数作为参数，这个参数可以传入两个参数：resolve，reject，分别表示异步操作执行成功后的回调函数和异步操作执行失败后的回调函数。其实这里用“成功”和“失败”来描述并不准确，按照标准来讲，resolve 是将 Promise 的状态置为 fullfiled，reject 是将 Promise 的状态置为 rejected。不过在我们开始阶段可以先这么理解，后面再细究概念。
* 
```js
var p = new Promise function (resolve, reject){
  // 做一些异步操作
  setTimeout(function() {
    var num = Math.ceil(Math.random()*10);// 生成 1-10 的随机数
    if(num<=5){
      resolve(num);
    }
    else{
      reject('数字太大了');
    }
  }, 2000);
});
```

new 了一个对象，并没有调用它，我们传进去的函数就已经执行了，这是需要注意的一个细节，所以我们用 Promise 的时候一般是包在一个函数中

* 执行这个函数我们得到了一个 Promise 对象，其原型上有 then、catch 方法

* then 接收两个参数，注册当 resolve（成功)/reject（失败) 的回调函数
```js
// onFulfilled 是用来接收 promise 成功的值
// onRejected 是用来接收 promise 失败的原因
promise.then(onFulfilled, onRejected);
```
  - then 方法中如果直接 return 数据，会包装成 promise
* catch 用来指定 reject 的回调。而且在执行 resolve 的回调（也就是上面 then 中的第一个参数）时，如果抛出异常了（代码出错了），那么并不会报错卡死 js，而是会进到这个 catch 方法中
* all 的用法
  - 提供了并行执行异步操作的能力，并且在所有异步操作执行完后才执行回调
* race 的用法
  - 谁跑的快，以谁为准执行回调

## 术语
* Promise
  - promise 是一个拥有 then 方法的对象或函数，其行为符合本规范；
* thenable
  - 是一个定义了 then 方法的对象或函数，文中译作“拥有 then 方法”；
* 值（value）
  - 指任何 JavaScript 的合法值（包括 undefined , thenable 和 promise）；
* 异常（exception）
  - 是使用 throw 语句抛出的一个值。
* 拒绝原因（reason）
  - 表示一个 promise 的拒绝原因。

## 要求
 
### Promise 的状态

一个 Promise 的当前状态必须为以下三种状态中的一种：等待态（Pending）、完成态（Fulfilled）和拒绝态（Rejected）。

* 等待态（Pending）
  - 处于等待态时，promise 需满足以下条件：
    * 可以迁移至完成态或拒绝态
* 完成态（Fulfilled）
  - 处于完成态时，promise 需满足以下条件：
    * 不能迁移至其他任何状态
    * 必须拥有一个不可变的终值
* 拒绝态（Rejected）
  - 处于拒绝态时，promise 需满足以下条件：
    * 不能迁移至其他任何状态
    * 必须拥有一个不可变的据因

这里的不可变指的是恒等（即可用 === 判断相等），而不是意味着更深层次的不可变（译者注： 盖指当 value 或 reason 不是基本值时，只要求其引用地址相等，但属性值可被更改）。

### Then 方法

一个 promise 必须提供一个 then 方法以访问其当前值、终值和据因。

promise 的 then 方法接受两个参数：

promise.then(onFulfilled, onRejected)

* 参数可选
  - onFulfilled 和 onRejected 都是可选参数。
  - 如果 onFulfilled 不是函数，其必须被忽略
  - 如果 onRejected 不是函数，其必须被忽略

注：如果我们只想传 onRejected 而不想传 onFulfilled，可以这么写：pormise.then(null, onRejected)

* onFulfilled 特性
  - 如果 onFulfilled 是函数：
    * 当 promise 执行结束后其必须被调用，其第一个参数为 promise 的终值
    * 在 promise 执行结束前其不可被调用
    * 其调用次数不可超过一次
* onRejected 特性
  - 如果 onRejected 是函数：
    * 当 promise 被拒绝执行后其必须被调用，其第一个参数为 promise 的据因
    * 在 promise 被拒绝执行前其不可被调用
    * 其调用次数不可超过一次

* 调用时机

onFulfilled 和 onRejected 只有在执行环境堆栈仅包含平台代码时才可被调用 注 1

* 调用要求

onFulfilled 和 onRejected 必须被作为函数调用（即没有 this 值）注 2

注：也就是说，我们在 promise 中就别用 this 了。

* 多次调用

then 方法可以被同一个 promise 调用多次
  * 当 promise 成功执行时，所有 onFulfilled 需按照其注册顺序依次回调
  * 当 promise 被拒绝执行时，所有的 onRejected 需按照其注册顺序依次回调

注：这里解释了我们可以链式调用，promise.then().then().then()

* 返回

then 方法必须返回一个 promise 对象 注 3

promise2 = promise1.then(onFulfilled, onRejected);
 
注：这就是我们能够进行链式调用的原因，因为 then 方法返回的还是一个 promise 对象。

* 如果 onFulfilled 或者 onRejected 返回一个值 x ，则运行下面的 Promise 解决过程：[[Resolve]](promise2, x)
* 如果 onFulfilled 或者 onRejected 抛出一个异常 e ，则 promise2 必须拒绝执行，并返回拒因 e
* 如果 onFulfilled 不是函数且 promise1 成功执行， promise2 必须成功执行并返回相同的值
* 如果 onRejected 不是函数且 promise1 拒绝执行， promise2 必须拒绝执行并返回相同的据因

### Promise 解决过程

Promise 解决过程 是一个抽象的操作，其需输入一个 promise 和一个值，我们表示为 [[Resolve]](promise, x)，如果 x 有 then 方法且看上去像一个 Promise ，解决程序即尝试使 promise 接受 x 的状态；否则其用 x 的值来执行 promise 。

这种 thenable 的特性使得 Promise 的实现更具有通用性：只要其暴露出一个遵循 Promise/A+ 协议的 then 方法即可；这同时也使遵循 Promise/A+ 规范的实现可以与那些不太规范但可用的实现能良好共存。

运行 [[Resolve]](promise, x) 需遵循以下步骤：

* x 与 promise 相等

如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise

* x 为 Promise

如果 x 为 Promise ，则使 promise 接受 x 的状态 注 4：

* 如果 x 处于等待态， promise 需保持为等待态直至 x 被执行或拒绝
* 如果 x 处于完成态，用相同的值执行 promise
* 如果 x 处于拒绝态，用相同的据因拒绝 promise
注：这里就是解释我们链式调用 then 时，可以继续进行异步操作，只要在 onFulfilled 中继续返回一个 promise 对象即可。例如：
runAsync1()
.then(function(data){
    console.log(data);
    return runAsync2(); // 返回值为 promise 对象
})
.then(function(data){
    console.log(data);
    return runAsync3();
})

x 为对象或函数
如果 x 为对象或者函数：

把 x.then 赋值给 then 注 5
如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise
如果 then 是函数，将 x 作为函数的作用域 this 调用之。传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise:
如果 resolvePromise 以值 y 为参数被调用，则运行 [[Resolve]](promise, y)
如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
如果 resolvePromise 和 rejectPromise 均被调用，或者被同一参数调用了多次，则优先采用首次调用并忽略剩下的调用
如果调用 then 方法抛出了异常 e：
如果 resolvePromise 或 rejectPromise 已经被调用，则忽略之
否则以 e 为据因拒绝 promise
如果 then 不是函数，以 x 为参数执行 promise
如果 x 不为对象或者函数，以 x 为参数执行 promise
如果一个 promise 被一个循环的 thenable 链中的对象解决，而 [[Resolve]](promise, thenable) 的递归性质又使得其被再次调用，根据上述的算法将会陷入无限递归之中。算法虽不强制要求，但也鼓励施者检测这样的递归是否存在，若检测到存在则以一个可识别的 TypeError 为据因来拒绝 promise 注 6。

Promise 实现代码

function Promise (executor) {
  let self = this // 缓存当前 promise 实例对象
  self.status = 'pending' // Promise 当前的状态
  self.data = undefined  // Promise 的值

  // Promise resolve 时的回调函数集，
  // 因为在 Promise 结束之前有可能有多个回调添加到它上面
  self.onResolvedCallback = [] 
  
  // Promise reject 时的回调函数集，
  // 因为在 Promise 结束之前有可能有多个回调添加到它上面
  self.onRejectedCallback = [] 

  function resolve(value) {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(function() { // 异步执行所有的回调函数
      if (self.status === 'pending') {
        self.status = 'resolved'
        self.data = value
        for (var i = 0; i < self.onResolvedCallback.length; i++) {
          self.onResolvedCallback[i](value)
        }
      }
    })
  }

  function reject(reason) {
    setTimeout(function() { // 异步执行所有的回调函数
      if (self.status === 'pending') {
        self.status = 'rejected'
        self.data = reason
        for (var i = 0; i < self.onRejectedCallback.length; i++) {
          self.onRejectedCallback[i](reason)
        }
      }
    }
  }
   
  // 执行 executor 并传入相应的参数
  // new 的时候就已经调用了
  // 考虑到执行 executor 的过程中有可能出错，
  // 所以我们用 try/catch 块给包起来，
  // 并且在出错后以 catch 到的值 reject 掉这个 Promise
  try { 
    // 执行 executor
    executor(resolve, reject) 
  } catch(e) {
    reject(e)
  }
}

// then 方法接收两个参数，onResolved，onRejected，分别为 Promise 成功或失败后的回调
Promise.prototype.then = function(onResolved, onRejected) {
  var self = this
  var promise2

  // 根据标准，如果 then 的参数不是 function，则我们需要忽略它，此处以如下方式处理
  onResolved = typeof onResolved === 'function' ? onResolved : function(value) {return value}
  onRejected = typeof onRejected === 'function' ? onRejected : function(reason) {throw reason}

  if (self.status === 'resolved') {
    // 如果 promise1（此处即为 this/self) 的状态已经确定并且是 resolved，我们调用 onResolved
    // 因为考虑到有可能 throw，所以我们将其包在 try/catch 块里
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onResolved(self.data)
        if (x instanceof Promise) { 
          // 如果 onResolved 的返回值是一个 Promise 对象，
          // 直接取它的结果做为 promise2 的结果
          x.then(resolve, reject)
        }
        resolve(x) // 否则，以它的返回值做为 promise2 的结果
      } catch (e) {
        reject(e) // 如果出错，以捕获到的错误做为 promise2 的结果
      }
    })
  }

  if (self.status === 'rejected') {
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onRejected(self.data)
        if (x instanceof Promise) {
          x.then(resolve, reject)
        }
      } catch (e) {
        reject(e)
      }
    })
  }

  if (self.status === 'pending') {
    // 如果当前的 Promise 还处于 pending 状态，我们并不能确定调用 onResolved 还是 onRejected，
    // 只能等到 Promise 的状态确定后，才能确实如何处理。
    // 所以我们需要把我们的**两种情况**的处理逻辑做为 callback 放入 promise1（此处即 this/self) 的回调数组里
    // 逻辑本身跟第一个 if 块内的几乎一致，此处不做过多解释
    return promise2 = new Promise(function(resolve, reject) {
      self.onResolvedCallback.push(function(value) {
        try {
          var x = onResolved(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })

      self.onRejectedCallback.push(function(reason) {
        try {
          var x = onRejected(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })
    })
  }
}
// 为了下文方便，我们顺便实现一个 catch 方法
Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}

/*
resolvePromise 函数即为根据 x 的值来决定 promise2 的状态的函数
也即标准中的 [Promise Resolution Procedure](https://promisesaplus.com/#point-47)
x 为`promise2 = promise1.then(onResolved, onRejected)`里`onResolved/onRejected`的返回值
`resolve`和`reject`实际上是`promise2`的`executor`的两个实参，因为很难挂在其它的地方，所以一并传进来。
相信各位一定可以对照标准把标准转换成代码，这里就只标出代码在标准中对应的位置，只在必要的地方做一些解释
*/
function resolvePromise(promise2, x, resolve, reject) {
  var then
  var thenCalledOrThrow = false

  if (promise2 === x) { // 对应标准 2.3.1 节
    return reject(new TypeError('Chaining cycle detected for promise!'))
  }

  if (x instanceof Promise) { // 对应标准 2.3.2 节
    // 如果 x 的状态还没有确定，那么它是有可能被一个 thenable 决定最终状态和值的
    // 所以这里需要做一下处理，而不能一概的以为它会被一个“正常”的值 resolve
    if (x.status === 'pending') {
      x.then(function(value) {
        resolvePromise(promise2, value, resolve, reject)
      }, reject)
    } else { // 但如果这个 Promise 的状态已经确定了，那么它肯定有一个“正常”的值，而不是一个 thenable，所以这里直接取它的状态
      x.then(resolve, reject)
    }
    return
  }

  if ((x !== null) && ((typeof x === 'object') || (typeof x === 'function'))) { // 2.3.3
    try {

      // 2.3.3.1 因为 x.then 有可能是一个 getter，这种情况下多次读取就有可能产生副作用
      // 即要判断它的类型，又要调用它，这就是两次读取
      then = x.then 
      if (typeof then === 'function') { // 2.3.3.3
        then.call(x, function rs(y) { // 2.3.3.3.1
          if (thenCalledOrThrow) return // 2.3.3.3.3 即这三处谁选执行就以谁的结果为准
          thenCalledOrThrow = true
          return resolvePromise(promise2, y, resolve, reject) // 2.3.3.3.1
        }, function rj(r) { // 2.3.3.3.2
          if (thenCalledOrThrow) return // 2.3.3.3.3 即这三处谁选执行就以谁的结果为准
          thenCalledOrThrow = true
          return reject(r)
        })
      } else { // 2.3.3.4
        resolve(x)
      }
    } catch (e) { // 2.3.3.2
      if (thenCalledOrThrow) return // 2.3.3.3.3 即这三处谁选执行就以谁的结果为准
      thenCalledOrThrow = true
      return reject(e)
    }
  } else { // 2.3.4
    resolve(x)
  }
}