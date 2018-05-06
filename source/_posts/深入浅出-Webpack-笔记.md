---
title: 深入浅出 Webpack 笔记
date: 2018-02-08 13:52:39
tags:
---

## 1-1 前端的发展
### 模块化
* 模块化是指把一个复杂的系统分解到多个模块以方便编码。

* CommonJS
  - 概述
    * Node 应用由模块组成，采用 CommonJS 模块规范
    * 每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见
    * 如果想在多个文件分享变量，必须定义为 global 对象的属性，这样写法是不推荐的
    * CommonJS 规范规定，每个模块内部，module 变量代表当前模块。这个变量是一个对象，它的 exports 属性（即 module.exports）是对外的接口。加载某个模块，其实是加载该模块的 module.exports 属性。
    * require 方法用于加载模块。
    * 特点
      - 所有代码都运行在模块作用域，不会污染全局作用域。
      - 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
      - 模块加载的顺序，按照其在代码中出现的顺序。
      - 同步加载
  - module 对象
    * Node 内部提供一个 Module 构建函数。所有模块都是 Module 的实例。
    * 每个模块内部，都有一个 module 对象，代表当前模块。它有以下属性。
      - module.id 模块的识别符，通常是带有绝对路径的模块文件名。
      - module.filename 模块的文件名，带有绝对路径。
      - module.loaded 返回一个布尔值，表示模块是否已经完成加载。
      - module.parent 返回一个对象，表示调用该模块的模块。
      - module.children 返回一个数组，表示该模块要用到的其他模块。
      - module.exports 表示模块对外输出的值。
    * module.exports 属性
      - module.exports 属性表示当前模块对外输出的接口，其他文件加载该模块，实际上就是读取 module.exports 变量。
    * exports 变量
      - 为了方便，Node 为每个模块提供一个 exports 变量，指向 module.exports
      - 造成的结果是，在对外输出模块接口时，可以向exports对象添加方法。
      - 注意，不能直接将 exports 变量指向一个值，因为这样等于切断了 exports 与 module.exports 的联系。
      - 这意味着，如果一个模块的对外接口，就是一个单一的值，不能使用 exports 输出，只能使用 module.exports 输出。
  - AMD 规范与 CommonJS 规范的兼容性
    * CommonJS 规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。
    * AMD 规范则是非同步加载模块，允许指定回调函数。
    * 由于 Node.js 主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以 CommonJS 规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用 AMD 规范。
    * AMD规范使用define方法定义模块
  - require 命令
    * Node使用CommonJS模块规范，内置的require命令用于加载模块文件
    * 基本用法
      - require 命令的基本功能是，读入并执行一个 JavaScript 文件，然后返回该模块的 exports 对象。如果没有发现指定模块，会报错。
      - 如果模块输出的是一个函数，那就不能定义在 exports 对象上面，而要定义在 module.exports 变量上面。
    * 加载规则
      - require 命令用于加载文件，后缀名默认为。js。
      - 根据参数的不同格式，require命令去不同路径寻找模块文件。
      1. 如果参数字符串以“/”开头，则表示加载的是一个位于绝对路径的模块文件。比如，require('/home/marco/foo.js') 将加载 /home/marco/foo.js。
      2. 如果参数字符串以“./”开头，则表示加载的是一个位于相对路径（跟当前执行脚本的位置相比）的模块文件。比如，require('./circle') 将加载当前脚本同一目录的 circle.js。
      3. 如果参数字符串不以“./“或”/“开头，则表示加载的是一个默认提供的核心模块（位于 Node 的系统安装目录中），或者一个位于各级 node_modules 目录的已安装模块（全局安装或局部安装）
      4. 如果参数字符串不以“./“或”/“开头，而且是一个路径，比如 require('example-module/path/to/file')，则将先找到 example-module 的位置，然后再以它为参数，找到后续路径。
      5. 如果指定的模块文件没有发现，Node 会尝试为文件名添加。js、.json、.node 后，再去搜索。.js 件会以文本格式的 JavaScript 脚本文件解析，.json 文件会以 JSON 格式的文本文件解析，.node 文件会以编译后的二进制文件解析。
      6. 如果想得到 require 命令加载的确切文件名，使用 require.resolve() 方法。
    * 目录的加载规则
      - 在目录中放置一个 package.json 文件，并且将入口文件写入 main 字段
      - require 发现参数字符串指向一个目录以后，会自动查看该目录的 package.json 文件，然后加载 main 字段指定的入口文件。如果 package.json 文件没有 main 字段，或者根本就没有 package.json 文件，则会加载该目录下的 index.js 文件或 index.node 文件。
    * 模块的缓存
      - 第一次加载某个模块时，Node 会缓存该模块。以后再加载该模块，就直接从缓存取出该模块的 module.exports 属性。
      - 所有缓存的模块保存在 require.cache 之中，如果想删除模块的缓存，delete require.cache[moduleName];
    * 模块的循环加载
      - 如果发生模块的循环加载，即 A 加载 B，B 又加载 A，则 B 将加载 A 的不完整版本。
  - 模块的加载机制
    * CommonJS 模块的加载机制是，输入的是被输出的值的拷贝。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。

### AMD
* AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。
* 采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行
* AMD 也采用 require() 语句加载模块，但是不同于 CommonJS，它要求两个参数 require([module], callback);

### CMD
* CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。
* CMD 则是依赖就近，用的时候再 require
```js
// CMD
define(function(require, exports, module) { 
  var a = require('./a')
  a.doSomething() // 此处略去 100 行
  var b = require('./b') // 依赖可以就近书写
  b.doSomething()   // ... 
})

// AMD 默认推荐的是
define(['./a', './b'], function(a, b) {
  // 依赖必须一开始就写好
  a.doSomething()
  b.doSomething()
  // 此处略去 100 行
  // ...
})
```

### ES6 模块化

* 何为模块？
  - 模块的真实力量是按需导出与导入代码的能力，而不用将所有内容放在同一
  个文件内
  - 特点
    1. 模块代码自动运行在严格模式下，并且没有任何办法跳出严格模式
    2. 在模块的顶级作用域创建的变量，不会被自动添加到共享的全局作用域，它们只会在模
    块顶级作用域的内部存在；
    3. 模块顶级作用域的 this 值为 undefined ；
    4. 模块不允许在代码中使用 HTML 风格的注释（ 这是 JS 来自于早期浏览器的历史遗留特性） ；
    5. 对于需要让模块外部代码访问的内容，模块必须导出它们；
    6. 允许模块从其他模块导入绑定。
* 基本的导出与导出
  - 最简单方法就是将 export 放置在任意变量、函数或类声明之前，从模块中将它们公开出去，
  - import{} 导入
  - 完全导入一个模块：import * as example from "./example.js";
  - 无论你对同一个模块使用了多少次 import 语句，该模块都只会被执行一次。在导出模块的代码执行之后，已被实例化的模块就被保留在内存中，并随时都能被其他import 所引用
  - export 与 import 都有一个重要的限制，那就是它们必须被用在其他语句或表达式的外部。
* 重命名导出与导入
  - 使用 as 关键字来指定新的名称
  - 
  ```js
  function sum(num1, num2) {
    return num1 + num2;
  }
  export { sum as add };
  // 
  import { add as sum } from "./example.js";
  ```
* 模块的默认值
```js
// 1
export default function(num1, num2) {
  return num1 + num2;
}
// 2
function sum(num1, num2) {
  return num1 + num2;
}
export default sum;
// 3
export { sum as default };

import sum from "./example.js";


export let color = "red";
export default function(num1, num2) {
  return num1 + num2;
}
// 你可以像下面这样使用 import 语句，来同时导入 color 以及作为默认值的函数：
// import sum, { color } from "./example.js";
// console.log(sum(1, 2)); // 3
// console.log(color);
```
