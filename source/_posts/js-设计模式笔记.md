---
title: js 设计模式笔记
date: 2018-03-07 16:25:52
tags:
---
# 基础知识
## 面向对象的 javascript
### 动态类型语言
* 编程语言按照**数据类型**大体可以分为两类，一类是静态类型语言，另一类是动态类型语言。
* 静态类型语言
  - 在编译时便已确定变量的类型
  - 好处
    * 在编译时就能发现类型不匹配的错误
    * 编译优化，提高程序执行速度
  - 缺点
    * 束缚性
    * 增加更多的代码
* 动态类型语言
  - 变量类型要到程序运行的时候，待变量被赋予某个值之后，才会具有某种类型
  - 优点
    * 代码数量少
  - 缺点
    * 无法保证变量的类型

### 鸭子类型
* 如果它走起路来像鸭子，叫起来也是鸭子，那么它就是鸭子
* 动态类型语言的面向对象设计中，鸭子类型的概念至关重要

### 多态
* 同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。
* 换句话说，给不同的对象发送同一个消息的时候，这些对象会根据这个消息分别给出不同的
反馈

## this、call和apply
### this
直接看 你不知道的js

call 是包装在 apply 上面的一颗语法糖
当使用 call 或者 apply 的时候，如果我们传入的第一个参数为 null，函数体内的 this 会指
向默认的宿主对象，在浏览器中则是 window

### 变量的生存周期
* 对于全局变量来说，全局变量的生存周期是永久的，除非我们主动销毁这个全局变量。
* 对于在函数内用 var 关键字声明的局部变量来说，当退出函数时，它们都会随着函数调用的结束而被销毁
*  f 返回了一个匿名函数的引用，它可以访问到 func()被调用时产生的环境，而局部变量 a 一直处在这个
环境里。既然局部变量所在的环境还能被外界访问，这个局部变量就有了不被销毁的理由。在这里产生了一个闭包结构，局部变量的生命看起来被延续了

### 高阶函数
* 高阶函数是指至少满足下列条件之一的函数：
  - 函数可以作为参数被传递；
    * 回调函数
  - 函数可以作为返回值输出
* 函数柯里化
  - currying 又称部分求值。一个 currying 的函数首先会接受一些参数，接受了这些参数之后，
  该函数并不会立即求值，而是继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保
  存起来。待到函数被真正需要求值的时候，之前传入的所有参数都会被一次性用于求值。
  - 
* 函数节流(throttle)
  - 函数被频繁调用的场景
    * window.onresize 事件
    * mousemove 事件
    * 上传进度
  - 函数节流的原理
    * 需要我们按时间段来忽略掉一些事件请求，比如确保在 500ms 内只打印一次
  - 函数节流的代码实现
  ```js
  function throttle (fn, wait) {
    let timer = null, isFirst = true, self = this

    return function () {
      if (isFirst) {
        fn.apply(self, arguments)
        isFirst = false
        return 
      }
      if (timer) {
        return false
      }
        timer = setTimeout(function() {
          clearTimeout(timer)
      timer = null
          fn.apply(self, arguments)
        }, wait)
    }
  }

  window.onscroll = throttle(function(){
    console.log( 1 );
  }, 2000 );
  ```
* 分时函数
```js
var timeChunk = function( ary, fn, count ) {
  var obj, t;
  var len = ary.length;
  var start = function() {
    for ( var i = 0; i < Math.min( count || 1, ary.length ); i++ ){
      var obj = ary.shift();
      fn( obj );
    }
  };
  return function() {
    t = setInterval(function() {
      if ( ary.length === 0 ){ // 如果全部节点都已经被创建好
        return clearInterval( t );
      }
      start();
    }, 200 ); // 分批执行的时间间隔，也可以用参数的形式传入
  };
};
```
* 惰性加载函数
```js
var addEvent = function( elem, type, handler ){
  if ( window.addEventListener ){
    addEvent = function( elem, type, handler ){
      elem.addEventListener( type, handler, false );
    }
  } else if ( window.attachEvent ){
    addEvent = function( elem, type, handler ){
      elem.attachEvent( 'on' + type, handler );
    }
  }
  addEvent( elem, type, handler );
};
```

# 第二部分 设计模式
* 单例模式的定义是： 保证一个类仅有一个实例，并提供一个访问它的全局访问点

### 避免全局变量
* 使用命名空间
  - 把一些方法定义为对象的属性
* 使用闭包封装私有变量