---
title: koa2 源码分析 (一)
date: 2017-12-06 14:18:02
tags:
---

koa2 启动一个 http server 的代码如下:
```js
const Koa = require('koa')
const app = new Koa()

app.use(async (ctx, next) => {
  console.log(`${ctx.request.url} ${ctx.request.method}`)
  await next()
})

app.use(async (ctx, next) => {
  console.log(`hello world !`)
  await next()
})

app.listen(3000, () => {
  console.log('server listening at 3000 port!')
})
```

知道怎么使用之后，我们对照着这个模板一点一点来拆分 koa2 整个源码
<!-- more -->
## Koa2 基本组成

Koa2 源码非常精简，只有四个文件：
  1. application.js：框架入口；负责管理中间件，以及处理请求
  2. context.js：context对象的原型，代理request与response对象上的方法和属性
  3. request.js：request对象的原型，提供请求相关的方法和属性
  4. response.js：response对象的原型，提供响应相关的方法和属性

## application.js

```js
// application.js
module.exports = class Application extends Emitter {
  /**
   * Initialize a new `Application`.
   * 初始化一个新的 Application
   * @api public
   */
  constructor() {
    super();
    this.proxy = false; // 是否信任 proxy header 参数，默认为 false(不懂)
    this.middleware = []; // app.use(middleware) 注册的中间件会保存在这里
    this.subdomainOffset = 2; // 子域默认偏移量，默认为 2 (不懂)
    this.env = process.env.NODE_ENV || 'development'; // 环境参数，默认为 NODE_ENV 或 ‘development’
    this.context = Object.create(context); // context模块，通过context.js创建
    this.request = Object.create(request); // request模块，通过request.js创建
    this.response = Object.create(response); //response模块，通过response.js创建
  }
  // ...
}
```

application.js 是 koa 的入口主要文件。我们可以看到， 在这个文件中最终暴露出来的是一个 class,
这个 class 继承自 EventEmitter 。

所以既然是暴露出来一个 class， 我们就需要用 new 操作符来调用，这也是我们模板中前两行代码
这么用的原因:
```js
const Koa = require('koa')
const app = new Koa() // 暴露出来的是一个类， 使用 class 必须使用new来调用
```

application.js除了上面的的构造函数外，还暴露了一些公用的api，比如两个常见的，一个是 listen，一个是use。

## use 函数
我们使用 use 函数的方法如下：
```js
app.user(async (ctx, next) => {
  console.log(`${ctx.request.url} ${ctx.request.method}`)
  await next()
})

app.user(async (ctx, next) => {
  console.log(`hello world !`)
  await next()
})
```

use 函数做的事很简单：注册一个中间件 fn，其实就是将 fn 放入middleware数组。

接下来看一下是怎么实现的:
```js
/**
 * Use the given middleware `fn`.
 *
 * Old-style middleware will be converted.
 *
 * @param {Function} fn
 * @return {Application} self
 * @api public
 */

use(fn) {
  // 首先判断传进来的阐述，传进来的不是一个函数，报错
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  // 判断这个函数是不是 generator
  // 因为 koa 后续的版本推荐使用 await/async 的方式处理异步
  // 所以会慢慢不支持 koa1 中的 generator
  // 所以不再推荐大家使用 generator
  if (isGeneratorFunction(fn)) {
    // 如果是 generator，控制台警告
    // 然后将函数进行包装
    deprecate('Support for generators will be removed in v3. ' +
              'See the documentation for examples of how to convert old middleware ' +
              'https://github.com/koajs/koa/blob/master/docs/migration.md');
    // 虽然嘴上警告，但身体还是很老实的嘛，:-D
    fn = convert(fn);
  }
  // 不知道这是干啥的
  debug('use %s', fn._name || fn.name || '-');
  // 将函数推入 middleware 这个数组，后面要依次调用里面的每一个中间件
  this.middleware.push(fn);
  // 保证链式调用
  return this;
}
```

## listen函数
listen 函数使用方法如下：
```js
app.listen(3000, () => {
  console.log('server listening at 3000 port!')
})
```

源码：
```js
// application.js
/**
 * Shorthand for:
 *
 *    http.createServer(app.callback()).listen(...)
 *
 * @param {Mixed} ...
 * @return {Server}
 * @api public
 */
listen(...args) {
  debug('listen'); // 暂时不用管
  // 首先 this.callback 方法会返回一个函数作为http.createServer的回调函数
  // 然后 serve 进行监听。
  const server = http.createServer(this.callback());
  return server.listen(...args);
}
```

这里我们需要一点前置知识，用 node 创建一个 http server 很简单

```js
const http = require('http')
const server = http.createServer((req, res) => {
})
server.listen(3000, () => {
  console.log('server listening at 3000 port!')
})
```
koa2 就是在基于原生 node 的东西封装了一下

app.listen(params) 调用的时候，首先会调用 this.callback()， 然后把它 return 出来的值，
作为 http.createServer 的回调函数放到原生的 http.createServer 中。
因为 http.createServer 接受一个函数作为参数，所以 this.callback()
肯定会返回一个函数

然后进行监听。我们已经知道，http.createServer的回调函数接收两个参数：req和res

下面来看this.callback的实现：

## callback 函数

```js
// application.js
callback() {
  // 使用了koa-compose
  // 把所有 middleware 进行了组合，然后返回了一个函数
  // fn 接受两个参数 (context, next)
  const fn = compose(this.middleware);
  // 暂时先不管
  if (!this.listeners('error').length) this.on('error', this.onerror);

  // 我们说过，this.callback 会返回一个函数
  // 这个函数会被放到 http.createServer 中，作为其回调函数
  // 每当有一个 http 请求过来就会执行这个回调函数
  // handleRequest 就是我们需要 return 出去的函数
  // http.createServer 的回调函数接受两个参数 req res
  const handleRequest = (req, res) => {
    // 首先，当一个请求走到这里，肯定是成功的，所以把http response code默认设置为404
    res.statusCode = 404;
    // 接着利用 createContext 函数把node返回的req和res进行了封装创建出 context
    // createContext 是一个对象，上面挂载了很多我们想要的信息
    const ctx = this.createContext(req, res);
    const onerror = err => ctx.onerror(err);
    const handleResponse = () => respond(ctx);
    onFinished(res, onerror);
    return fn(ctx).then(handleResponse).catch(onerror);
  };

  return handleRequest;
}
```

## compose 函数

前面知道，compose 是这样用的：
```js
const fn = compose(this.middleware);
```

this.middleware 是一个数组，存放着我们所有的中间件。compose 函数接受这个数组，
把所有 middleware 进行了组合，然后返回了一个函数

我们来看一下 koa-compose 的代码：
```js
// compose index.js
// https://github.com/koajs/compose/blob/master/index.js

'use strict'

/**
 * Expose compositor.
 */

// 导出一个函数
module.exports = compose

/**
 * Compose `middleware` returning
 * a fully valid middleware comprised
 * of all those which are passed.
 *
 * @param {Array} middleware
 * @return {Function}
 * @api public
 */

function compose (middleware) {
  // 传入的 middleware 必须是一个数组, 否则报错
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  // 循环遍历传入的 middleware， 每一个元素都必须是函数，否则报错
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */
  // 正如我们前面说的， compose 函数会返回一个函数
  // 第一次只需要看到这里就好了，记住，compose 会返回一个函数
  // 这个函数接受两个参数 context, next
  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      // 如果是最后一个中间件
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, function next () {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}

```

我们来调试一下

```js

function compose (middleware) {
  // 传入的 middleware 必须是一个数组, 否则报错
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  // 循环遍历传入的 middleware， 每一个元素都必须是函数，否则报错
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */
  // 正如我们前面说的， compose 函数会返回一个函数
  // 记住，compose 会返回一个函数，这个函数接受两个参数 context, next
  return function (context, next) {
    // last called middleware #
    let index = -1
    debugger
    return dispatch(0)
    function dispatch (i) {
      debugger
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      // 已经取完了所有中间件
      // 这个时候还会进来一次，然后把 fn 变成 next
      // 在这里面 next 没有传，所以 fn = next = undefined
      // 直接 resolve 掉
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, function next () {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}

const ctx = {
  onerror: (err) => {
    console.log(err, 'err')
  }
}

const middleware = [async (ctx, next) => {
  console.log(ctx, next, '1')
  await next()
}, async (ctx, next) => {
  console.log(ctx, next, '2')
  await next()
}, async (ctx, next) => {
  console.log(ctx, next, '3')
  await next()
}]

// fn 接受两个参数，一个ctx， 一个 next
const fn = compose(middleware)

const onerror = err => ctx.onerror(err)
const handleResponse = () => console.log(ctx)

fn(ctx).then(handleResponse).catch(onerror);

```
我们写了三个中间件，放到了 middleware 中， 然后在 compose 函数里面打了两个断点
把上面的代码复制，然后在浏览器的 Console 中执行一下

fn(ctx) 一执行，第一次 index = -1，然后被赋值为 inedx = 0,里面的 fn 是middleware[i],
也就是取到了第一个中间件

然后执行第一个中间件，同时把它resolve，因为 dispatch 会返回一个 Promise， 所以可以用await

从源码中我们可以看出来
1. 中间件的书写顺序是很重要的
2. 如果中间件中没有 await next ，那么函数直接就退出了，不会继续递归调用

## createContext 函数
createContext 的调用只出现了一次，用法如下:
```js
const handleRequest = (req, res) => {
  // 接着利用createContext函数把node的req和res进行了封装创建出context，
  // ctx 是一个对象
  const ctx = this.createContext(req, res);
  return this.handleRequest(ctx, fn);
};
```
也就是创建了一个 ctx 对象

我们看一下代码的实现
```js
/**
 * Initialize a new context.
 *
 * @api private
 */

createContext(req, res) {
  // createContext 接受 node的 req 和 res
  // 创建 ctx 对象
  const context = Object.create(this.context);
  // 创建 request,挂载在 ctx 上
  const request = context.request = Object.create(this.request);
  // 创建 response,挂载在 ctx 上
  const response = context.response = Object.create(this.response);

  context.app = request.app = response.app = this;
  context.req = request.req = response.req = req;
  context.res = request.res = response.res = res;
  request.ctx = response.ctx = context;
  request.response = response;
  response.request = request;
  context.originalUrl = request.originalUrl = req.url;
  context.cookies = new Cookies(req, res, {
    keys: this.keys,
    secure: request.secure
  });
  request.ip = request.ips[0] || req.socket.remoteAddress || '';
  context.accept = request.accept = accepts(req);
  context.state = {};
  return context;
}
```

## respond 函数
respond 的用法是这样的：

```js
const handleResponse = () => respond(ctx);
```

源码：
```js
function respond(ctx) {
  // allow bypassing koa
  if (false === ctx.respond) return;

  const res = ctx.res;
  if (!ctx.writable) return;

  let body = ctx.body;
  const code = ctx.status;

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null;
    return res.end();
  }

  if ('HEAD' == ctx.method) {
    if (!res.headersSent && isJSON(body)) {
      ctx.length = Buffer.byteLength(JSON.stringify(body));
    }
    return res.end();
  }

  // status body
  if (null == body) {
    body = ctx.message || String(code);
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  }

  // responses
  if (Buffer.isBuffer(body)) return res.end(body);
  if ('string' == typeof body) return res.end(body);
  if (body instanceof Stream) return body.pipe(res);

  // body: json
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}
```
