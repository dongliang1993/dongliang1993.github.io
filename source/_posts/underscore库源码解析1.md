---
title: underscore 源码解析1 -- 骨架剖析
date: 2018-02-02 12:35:04
tags:
---

这是我们源码分析的第一章，主要是从整体的角度来剖析一个库的架构，不会拘泥于细枝末节。大体如下：

<!-- more -->

```js
(function(){
  // Baseline setup
  // 基本设置、配置
  // --------------

  // Establish the root object, `window` in the browser, or `exports` on the server.
  // 将 this 赋值给局部变量 root
  // root 的值, 客户端为 `window`, 服务端(node) 中为 `exports`
  var root = this;

  // Save the previous value of the `_` variable.
  // 将原来全局环境中的变量 `_` 赋值给变量 previousUnderscore 进行缓存
  // 主要是为了防止冲突
  // 在后面的 noConflict 方法中有用到
  var previousUnderscore = root._;

  // Save bytes in the minified (but not gzipped) version:
  // 缓存变量, 便于压缩代码
  // 此处「压缩」指的是压缩到 min.js 版本
  // 而不是 gzip 压缩
  var ArrayProto = Array.prototype,
      ObjProto = Object.prototype,
      FuncProto = Function.prototype;

  // Create quick reference variables for speed access to core prototypes.
  // 缓存变量, 便于压缩代码
  // 同时可减少在原型链中的查找次数(提高代码效率)
  var
    push             = ArrayProto.push,
    slice            = ArrayProto.slice,
    toString         = ObjProto.toString,
    hasOwnProperty   = ObjProto.hasOwnProperty;

  // All **ECMAScript 5** native function implementations that we hope to use
  // are declared here.
  // ES5 原生方法, 如果浏览器支持, 则 underscore 中会优先使用
  var
    nativeIsArray      = Array.isArray,
    nativeKeys         = Object.keys,
    nativeBind         = FuncProto.bind,
    nativeCreate       = Object.create;
	
  // Naked function reference for surrogate-prototype-swapping.
  var Ctor = function(){};

  // Create a safe reference to the Underscore object for use below.
  // 核心函数
  // `_` 其实是一个支持无 new 调用的构造函数（思考 jQuery 的无 new 调用）
  // 将传入的参数（实际要操作的数据）赋值给 this._wrapped 属性
  // OOP 调用时，_ 相当于一个构造函数 
  // each 等方法都在该构造函数的原型链上
  // _([1, 2, 3]).each(alert)
  // _([1, 2, 3]) 相当于无 new 构造了一个新的对象
  // 调用了该对象的 each 方法，该方法在该对象构造函数的原型链上
  var _ = function(obj) {
    // 以下均针对 OOP 形式的调用
    // 如果是非 OOP 形式的调用，不会进入该函数内部

    // 如果 obj 已经是 `_` 函数的实例，则直接返回 obj
    if (obj instanceof _)
      return obj;

    // 如果不是 `_` 函数的实例
    // 则调用 new 运算符，返回实例化的对象
    if (!(this instanceof _))
      return new _(obj);

    // 将 obj 赋值给 this._wrapped 属性
    this._wrapped = obj;
  };
  
  
  // Export the Underscore object for **Node.js**, with
  // backwards-compatibility for the old `require()` API. If we're in
  // the browser, add `_` as a global object.
  // 将上面定义的 `_` 局部变量赋值给全局对象中的 `_` 属性
  // 即客户端中 window._ = _
  // 服务端(node)中 exports._ = _
  // 同时在服务端向后兼容老的 require() API
  // 这样暴露给全局后便可以在全局环境中使用 `_` 变量(方法)
  if (typeof exports !== 'undefined') {
    if (typeof module !== 'undefined' && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    // 如果 exports 未定义，也就是说是在浏览器中
    root._ = _;
  }

  // Current version.
  // 当前 underscore 版本号
  _.VERSION = '1.8.3';

	/*各种内部变量、函数*/
	//...
	
	/*各种公开的静态api,如：*/
	_.each = function(){
		//TODO
	};
	_.first = function(){
		//TODO
	};
	_.bind = function(){
		//TODO
	};
	_.keys = function(){
		//TODO
	};
	_.noConfict = function(){
		//TODO
	};
	_.chain = function(){
		//TODO
	};
  
  //...
  //...
  //...
  
  // ***实例方法***

	// Extracts the result from a wrapped and chained object.
  // 一个包装过(OOP)并且链式调用的对象
  // 用 value 方法获取结果
  // _(obj).value === obj?
  _.prototype.value = function() {
    return this._wrapped;
  };

  // Provide unwrapping proxy for some methods used in engine operations
  // such as arithmetic and JSON stringification.
  _.prototype.valueOf = _.prototype.toJSON = _.prototype.value;

  _.prototype.toString = function() {
    return '' + this._wrapped;
  };
  // ***实例方法结束***

  // AMD registration happens at the end for compatibility with AMD loaders
  // that may not enforce next-turn semantics on modules. Even though general
  // practice for AMD registration is to be anonymous, underscore registers
  // as a named module because, like jQuery, it is a base library that is
  // popular enough to be bundled in a third party lib, but not be part of
  // an AMD load request. Those cases could generate an error when an
  // anonymous define() is called outside of a loader request.
  // 兼容 AMD 规范
  if (typeof define === 'function' && define.amd) {
    define('underscore', [], function() {
      return _;
    });
  }
).call(this));
```

1. 外边调用了 call ，把 this 绑定。然后用局部变量 root 来接收 this。this 根据代码使用的平台不一样也是不同的。比如， 客户端为 'window', 服务端(node) 中为 'exports'
2. 如果之前的 this 上有 _ 这个变量, 怎么防止冲突？代码里面用 previousUnderscore 接收了 root._，后面会进行处理冲突
3. 接下来要开始构造我们自己的变量 _ ，这是最关键的一步，所有的方法都是挂载在这个变量上的。**_ 其实是一个构造函数**，支持无 new 调用（思考 jQuery 的无 new 调用）。一般有两种使用方法
  * _.each([1,2,3], alert)
  * _([1, 2, 3]).each(alert) (OOP 调用)
通常我们会使用第二种方式。我们看一下代码实现
```js

  // Create a safe reference to the Underscore object for use below.
  // 核心函数
  // `_` 其实是一个构造函数
  // 支持无 new 调用（思考 jQuery 的无 new 调用）
  // 将传入的参数（实际要操作的数据）赋值给 this._wrapped 属性
  // OOP 调用时，_ 相当于一个构造函数 
  // each 等方法都在该构造函数的原型链上
  // _([1, 2, 3]).each(alert)
  // _([1, 2, 3]) 相当于无 new 构造了一个新的对象
  // 调用了该对象的 each 方法，该方法在该对象构造函数的原型链上
  var _ = function(obj) {
    // 以下均针对 OOP 形式的调用
    // 如果是非 OOP 形式的调用，不会进入该函数内部

    // 如果 obj 已经是 `_` 函数的实例，则直接返回 obj
    if (obj instanceof _)
      return obj;

    // 如果不是 `_` 函数的实例
    // 则调用 new 运算符，返回实例化的对象
    if (!(this instanceof _))
    
      return new _(obj);

    // 将 obj 赋值给 this._wrapped 属性
    this._wrapped = obj;
  };
```
4. _ 这个变量定义好了，接下来我们要挂载到 this 上
```js
// Export the Underscore object for **Node.js**, with
// backwards-compatibility for the old `require()` API. If we're in
// the browser, add `_` as a global object.
// 将上面定义的 `_` 局部变量赋值给全局对象中的 `_` 属性
// 即客户端中 window._ = _
// 服务端(node)中 exports._ = _
// 同时在服务端向后兼容老的 require() API
// 这样暴露给全局后便可以在全局环境中使用 `_` 变量(方法)
if (typeof exports !== 'undefined') {
  // node 环境
  if (typeof module !== 'undefined' && module.exports) {
    exports = module.exports = _;
  }
  exports._ = _;
} else {
  // 如果 exports 未定义，也就是说是在浏览器中
  root._ = _;
}
```