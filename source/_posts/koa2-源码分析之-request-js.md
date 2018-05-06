---
title: koa2 源码分析之 request.js
date: 2017-12-07 14:49:50
tags:
---
这篇文章我们主要分析一下 request.js 文件的源码

## 概览

```js
// request.js
module.exports = {
  ...
}
```

将代码收起来，我们可以看到，request.js 中导出的是一个对象。

为什么会导出一个对象呢？我们先看一下在 application.js 中是如何使用的。

```js
// application.js
// 省略了无关紧要的代码
// ...
const request = require('./request');
// ...
module.exports = class Application extends Emitter {
  constructor() {
    super();
    // ...
    this.context = Object.create(context); // context模块，通过context.js创建
    this.request = Object.create(request); // request模块，通过request.js创建
    this.response = Object.create(response); //response模块，通过response.js创建
  }
}

````

可以看到，在 Application 这个类的构造函数中，request 是当做一个参数传给 Object.create
这个函数的。

先讲一下 Object.create 这个方法，首先看一下 MDN 对应的文档
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create

Object.create() 方法会使用指定的原型对象及其属性去创建一个新的对象。用法如下：
```js
Object.create(proto [, propertiesObject ])
```

1. proto 为新创建对象的原型对象，设置为null可创建没有原型的空对象。
2. 如果 proto 参数不是 null 或一个对象，则抛出一个 TypeError 异常。
2. propertiesObject 可选。如果没有指定为 undefined，则是要添加到新创建对象的可枚举属性
（即其自身定义的属性，而不是其原型链上的枚举属性）对象的属性描述符以及相应的属性名称。
这些属性对应Object.defineProperties()的第二个参数。


简单来说，就是创建了一个对象，然后这个对象的原型就是你进去的那个参数

## 简单包装

其实 request.js 这部分代码没有什么很复杂的地方，主要是把我们常用的属性读取和赋值包装成了 getter 和 setter
附上 MDN 对应的文档 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/get

举个例子，比如我们想查看请求的 URL，我们会用 ctx.resquest.url 来查看，也可以用 ctx.url。而 ctx.url
就是被包装成了 getter 和 setter：
```js
get url() {
  return this.req.url;
},

/**
 * Set request URL.
 *
 * @api public
 */

set url(val) {
  this.req.url = val;
},
```

无非就是暴露出来一下语法糖，方便开发而已
