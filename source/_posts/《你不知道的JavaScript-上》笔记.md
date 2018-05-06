---
title: 《你不知道的JavaScript-上》笔记
date: 2018-01-30 10:57:59
tags: "js"
---

## 作用域和闭包

### 作用域是什么
* 需要一套设计良好的规则来存储变量， 并且之后可以方便地找到这些变量。这套规则被称为**作用域**
* 编译原理
  - 在传统编译语言的流程中， 程序中的一段源代码在执行之前会经历三个步骤， 统称为“编译”
    * 分词/词法分析（ Tokenizing/Lexing）
    * 解析/语法分析（ Parsing）
      - 这个过程是将词法单元流（ 数组） 转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树。 这个树被称为“ 抽象语法树”（Abstract Syntax Tree， AST）。
    * 代码生成
      - 将 AST 转换为可执行代码的过程称被称为代码生成。 
* 理解作用域
  - 变量的赋值操作会执行两个动作， 首先编译器会在当前作用域中声明一个变量（ 如果之前没有声明过）， 然后在运行时引擎会在作用域中查找该变量， 如果能够找到就会对它赋值。
  - LHS 和 RHS
    * “L” 和“R” 的含义， 它们分别代表左侧和右侧
    * 当变量出现在赋值操作的左侧时进行 LHS 查询， 出现在右侧时进行 RHS 查询。
    * 如果查找的目的是对变量进行赋值， 那么就会使用 LHS 查询； 如果目的是获取变量的值， 就会使用 RHS 查询。
* 作用域嵌套
  - 当一个块或函数嵌套在另一个块或函数中时， 就发生了作用域的嵌套。 
  -  引擎从当前的执行作用域开始查找变量， 如果找不到，就向上一级继续查找。 当抵达最外层的全局作用域时， 无论找到还是没找到， 查找过程都会停止


### 词法作用域
* 作用域共有两种主要的工作模型：词法作用域、动态作用域
* 词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。 
* 词法阶段
  - 作用域查找会在找到第一个匹配的标识符时停止。 在多层的嵌套作用域中可以定义同名的标识符， 这叫作“ 遮蔽效应”（ 内部的标识符“ 遮蔽” 了外部的标识符）
  - 无论函数在哪里被调用， 也无论它如何被调用， 它的词法作用域都只由函数被声明时所处的位置决定。
* 欺骗词法
  - 欺骗词法作用域会导致性能下降
  - eval
    * JavaScript 中的 eval(..) 函数可以接受一个字符串为参数， 并将其中的内容视为好像在书写时就存在于程序中这个位置的代码。 换句话说， 可以在你写的代码中用程序生成代码并运行， 就好像代码是写在那个位置的一样。
  - with
    * 

### 函数作用域和块作用域
* 函数中的作用域
  - 每声明一个函数都会为其自身创建一个作用域
  - 函数作用域的含义是指， 属于这个函数的全部变量都可以在整个函数的范围内使用及复用（ 事实上在嵌套的作用域中也可以使用）
* 隐藏内部实现
  - 规避冲突
    * 1. 全局命名空间
    * 2. 模块管理
* 函数作用域
  - 立即执行函数表达式（IIFE）

### 块作用域
* 变量的声明应该距离使用的地方越近越好， 并最大限度地本地化
* with
* try/catch：  try/catch 的 catch 分句会创建一个块作用域， 其中声明的变量仅在 catch 内部有效
* let
* const

### 提升
* 包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理
* 当你看到 var a = 2; 时， 可能会认为这是一个声明。 但 JavaScript 实际上会将其看成两个
声明： var a; 和 a = 2;。 第一个定义声明是在编译阶段进行的。 第二个赋值声明会被留在
原地等待执行阶段。
* 只有声明本身会被提升， 而赋值或其他运行逻辑会留在原地
* 每个作用域都会进行提升操作
* 函数声明会被提升， 但是函数表达式却不会被提升。
* 即使是具名的函数表达式， 名称标识符在赋值之前也无法在所在作用域中使用

### 函数优先
* 函数声明和变量声明都会被提升。 但是一个值得注意的细节（ 这个细节可以出现在有多个
“ 重复” 声明的代码中） 是函数会首先被提升， 然后才是变量


## 作用域闭包
* 当函数可以记住并访问所在的词法作用域时， 就产生了闭包， 即使函数是在当前词法作用
域之外执行








## 关于 this
* 为什么要用this
  - this 提供了一种更优雅的方式来隐式“传递” 一个对象引用, 复用代码
* 误解
  - this 不是指向函数自身
    * 从函数对象内部引用它自身(主要是用于递归或者解绑)
      - 函数名引用（匿名函数不可用）
      - arguments.callee
  - this 不是指函数的作用域
    *  this 在任何情况下都不指向函数的词法作用域。 在 JavaScript 内部， 作用域确实和对象类似， 可见的标识符都是它的属性。 但是作用域“ 对象” 无法通过 JavaScript代码访问， 它存在于 JavaScript 引擎内部
* this到底是什么
  - this 是在运行时进行绑定的， 并不是在编写时绑定， 它的上下文取决于函数调用时的各种条件。
  - this 的绑定和函数声明的位置没有任何关系，它指向什么完全取决于函数的调用方式。
  - 当一个函数被调用时， 会创建一个活动记录（ 有时候也称为执行上下文）。 这个记录会包含函数在哪里被调用（ 调用栈）、 函数的调用方法、 传入的参数等信息。 this 就是记录的其中一个属性， 会在函数执行的过程中用到。

## this 全面解析
* 调用位置
  - 调用位置就是函数在代码中被调用的位置（而不是声明的位置）
  - 分析调用栈（就是为了到达当前执行位置所调用的所有函数）
* 绑定规则
  * 默认绑定
    - 独立函数调, 也就是直接使用不带任何修饰的函数引用进行调用。可以把这条规则看作是无法应用其他规则时的默认规则。
    - this 指向
      * 指向全局对象
      * 如果使用严格模式（ strict mode）， 那么全局对象将无法使用默认绑定，this 会绑定到 undefined
      ```js
        function foo() {
          "use strict";
          console.log( this.a );
        }
        var a = 2;
        foo(); // 2

        function foo() {
          "use strict";
          console.log( this.a );
        }
        var a = 2;
        foo(); // Cannot read property 'a' of undefined
        
        // 虽然 this 的绑定规则完全取决于调用位置， 但是只
        // 有 foo() 运行在非 strict mode 下时， 默认绑定才能绑定到全局对象； 
        严格模式下与 foo()的调用位置无关：

        function foo() {
          console.log( this.a );
        }
        var a = 2;
        (function(){
          "use strict";
          foo(); // 2
        })();
      ```
  * 隐式绑定
    - 调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含
    - 当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象
    - 对象属性引用链中只有最顶层或者说最后一层会影响调用位置。
      ```js
        function foo() {
          console.log( this.a );
        }
        var obj2 = {
          a: 42,
          foo: foo
        };
        var obj1 = {
          a: 2,
          obj2: obj2
        };
        obj1.obj2.foo(); // 42
      ```
    - 隐式丢失
      * 被隐式绑定的函数会丢失绑定对象， 也就是说它会应用默认绑定， 从而把 this 绑定到全局对象或者 undefined 上， 取决于是否是严格模式
        ```js
          function foo() {
            console.log( this.a );
          }

          function doFoo(fn) {
            // fn 其实引用的是 foo

            fn(); // <-- 调用位置！
          }

          var obj = {
            a: 2,
            foo: foo
          };
          var a = "oops, global"; // a 是全局对象的属性

          doFoo( obj.foo ); // "oops, global"
          // 参数传递其实就是一种隐式赋值， 因此我们传入函数时也会被隐式赋值， 
          // 所以结果和上一个例子一样
        ```
  * 显式绑定
    - 使用函数的 call(..) 和 apply(..) 方法
    - 如果你传入了一个原始值（ 字符串类型、 布尔类型或者数字类型） 来当作 this 的绑定对象， 这个原始值会被转换成它的对象形式（ 也就是 new String(..)、 new Boolean(..) 或者new Number(..)）。 这通常被称为“ 装箱”。
    - 硬绑定
      * 硬绑定的典型应用场景就是创建一个包裹函数(辅助函数)， 传入所有的参数并返回接收到的所有值
        ```js
        // 简单的辅助绑定函数

        function bind(fn, obj) {
          return function() {
            return fn.apply( obj, arguments );
          };
        }
        function foo() {
          console.log( this.a );
        }
        var obj = {
          a:2
        };
        var bar = function() {
          return foo.apply( obj, arguments )
        };
        bar(); // 2
        ```
      * 由于硬绑定是一种非常常用的模式， 所以在 ES5 中提供了内置的方法 Function.prototype.bind
  * new绑定
    -  在 JavaScript 中， 构造函数只是一些被 new 操作符调用的普通函数。
    - 实际上并不存在所谓的“ 构造函数”， 只有对于函数的“ 构造调用”
    - 使用 new 来调用函数， 或者说发生构造函数调用时， 会自动执行下面的操作。
      1. 创建（ 或者说构造） 一个全新的对象。
      2. 这个新对象会被执行 [[ 原型 ]] 连接。
      3. 这个新对象会绑定到函数调用的 this。
      4. 如果函数没有返回其他对象， 那么 new 表达式中的函数调用会自动返回这个新对象。
  * 优先级
    1. 函数是否在 new 中调用（ new 绑定）？ 如果是的话 this 绑定的是新创建的对象。
    var bar = new foo()
    2. 函数是否通过 call、 apply（ 显式绑定） 或者硬绑定调用？ 如果是的话， this 绑定的是指定的对象。
    var bar = foo.call(obj2)
    3. 函数是否在某个上下文对象中调用（ 隐式绑定） ？ 如果是的话， this 绑定的是那个上下文对象。
    var bar = obj1.foo()
    4. 如果都不是的话， 使用默认绑定。 如果在严格模式下， 就绑定到 undefined， 否则绑定到全局对象。
    var bar = foo()
  * 绑定例外
    - 如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、 apply 或者 bind， 这些值在调用时会被忽略， 实际应用的是默认绑定规则
      * 默认绑定规则会把 this 绑定到全局对象可能产生一些副作用
      * 一种“ 更安全” 的做法是传入一个特殊的对象（DMZ 对象）， 把 this 绑定到这个对象不会对你的程序产生任何副作用
      * Object.create(null) 和 {} 很像， 但是并不会创建 Object.prototype 这个委托， 所以它比 {}“ 更空”
    - 间接引用
      * 赋值表达式 p.foo = o.foo 的返回值是目标函数的引用， 因此调用位置是 foo() 而不是p.foo() 或者 o.foo()。 根据我们之前说过的， 这里会应用默认绑定
      ```js
        function foo() {
          console.log( this.a );
        }
        var a = 2;
        var o = { a: 3, foo: foo };
        var p = { a: 4 };
        o.foo(); // 3

        (p.foo = o.foo)(); // 2
      ```
    - 软绑定
      * 给默认绑定指定一个全局对象和 undefined 以外的值，实现和硬绑定相同的效果， 同时保留隐式绑定或者显式绑定修改 this 的能力。
        ```js
        if (!Function.prototype.softBind) {
          Function.prototype.softBind = function(obj) {
            var fn = this;
            // 捕获所有 curried 参数

            var curried = [].slice.call( arguments, 1 );
            var bound = function() {
              // 对指定的函数进行封装， 首先检查调用时的 this， 如果 this 绑定到全局对象
              // 或者 undefined 那就把指定的默认对象 obj 绑定到 this， 否则不会修改 this

              return fn.apply(
                (!this || this === (window || global)) ?
                obj : this,
                curried.concat.apply( curried, arguments )
              );
            };
            bound.prototype = Object.create( fn.prototype );
            return bound;
          };
        }
        ```
    - 箭头函数
      * 箭头函数不使用 this 的四种标准规则， 而是会继承外层函数调用的 this 绑定
      * 其实和 ES6 之前代码中的 self = this 机制一样。

## 对象
* 对象可以通过两种形式定义：声明（ 文字）形式和构造形式。
* 内置对象：例如 Object、 Array、 Function 和 RegExp。实际上只是一些内置函数，这些内置函数可以当作构造函数
* 在必要时语言会自动把字符串字面量转换成一个 String 对象， 也就是说你并不需要
显式创建一个对象。 (包装类型)
* 内容
  - 存储在对象容器内部的是属性的名称，它们就像指针（ 从技术角度来说就是引用） 一样，指向这些值真正的存储位置。
  - 属性名
    * 访问方法：属性访问和键访问
    * 属性名永远都是字符串。 如果你使用 string（ 字面量） 以外的其他值作为属性名， 那它首先会被转换为一个字符串。 即使是数字也不例外。(调用toString方法)
  - 可计算属性名
    * 使用 [] 包裹一个表达式来当作属性名
    ```js
      var prefix = "foo";
      var myObject = {
        [prefix + "bar"]:"hello",
        [prefix + "baz"]: "world"
      };
      myObject["foobar"]; // hello

      myObject["foobaz"]; // world
    ```
  - 属性与方法
    * 从技术角度来说， 函数永远不会“ 属于” 一个对象， 所以把对象内部引用的函数称为“ 方法” 似乎有点不妥。
    * someFoo 和 myObject.someFoo 只是对于同一个函数的不同引用， 并不能说明这个函数是特别的或者“ 属于” 某个对象。 
    * 即使你在对象的文字形式中声明一个函数表达式， 这个函数也不会“ 属于” 这个对象——它们只是对于相同函数对象的多个引用。
  - 数组
    * 数组也是对象， 所以虽然每个下标都是整数， 你仍然可以给数组添加属性
    * 可以看到虽然添加了命名属性（ 无论是通过 . 语法还是 [] 语法）， 数组的 length 值并未发生变化。
  - 复制对象
    * 浅复制、深复制
    * 对于 JSON 安全（ 也就是说可以被序列化为一个 JSON 字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象） 的对象来说， 有一种巧妙的复制方法：var newObj = JSON.parse( JSON.stringify( someObj ) );
    * Object.assign(..) 进行浅复制
  - 属性描述符
    * Object.getOwnPropertyDescriptor
      - 获取一个对象身上对应属性的描述符
      - 
      ```js
        var myObject = {
          a:2
        };
        Object.getOwnPropertyDescriptor( myObject, "a" );
        // {
        // value: 2,
        // writable: true,
        // enumerable: true,
        // configurable: true
        // }
      ```
    * Object.defineProperty(..)来添加一个新属性或者修改一个已有属性（ 如果它是 configurable） 并对特性进行设置。
      - 
        ```js
        var myObject = {};
        Object.defineProperty( myObject, "a", {
          value: 2,
          writable: true,
          configurable: true,
          enumerable: true
        } );
        ```
    * writable：决定是否可以修改属性的值
      - 设置为 false 对于属性值的修改静默失败（ silently failed）
    * Configurable: 属性是否可配置
      - 只要属性是可配置的，就可以使用 defineProperty(..) 方法来修改属性描述符
      - 把 configurable 修改成 false 是单向操作，尝试修改一个不可配置的属性描述符都会出错
      - configurable: false 还会禁止删除这个属性
    * Enumerable：属性是否可枚举
      - 如果把 enumerable 设置成 false， 这个属性就不会出现在枚举中， 虽然仍然可以正常访问它
  - 不变性
    * 所有的方法创建的都是浅不变形
    * 对象常量
      - 结合 writable:false 和configurable:false 就可以创建一个真正的常量属性（ 不可修改、重定义或者删除）
    * 禁止扩展
      - 如 果 你 想 禁 止 一 个 对 象 添 加 新 属 性 并 且 保 留 已 有 属 性， 可 以 使 用 Object.preventExtensions(..)
      - Object.preventExtensions(obj)
      - 密封
        * 不能添加新属性，也不能重新配置或者删除任何现有属性（ 虽然可以修改属性的值)
        * Object.seal(..)
        * 这个方法实际上会在一个现有对象上调用Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。
      - 冻结
        * 这个方法是你可以应用在对象上的级别最高的不可变性， 它会禁止对于对象本身及其任意直接属性的修改
        * Object.freeze(..) 
  - [[Get]]
    * 
      ```js
      var myObject = {
        a: 2
      };
      myObject.a; // 2
      ```
    * myObject.a 在 myObject 上实际上是实现了 [[Get]] 操作（ 有点像函数调用： [[Get]]()）。对象默认的内置 [[Get]] 操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值。然而， 如果没有找到名称相同的属性， 按照 [[Get]] 算法的定义会执行另外一种非常重要的行为。 我们会在第 5 章中介绍这个行为（ 其实就是遍历可能存在的 [[Prototype]] 链，也就是原型链）。如果无论如何都没有找到名称相同的属性， 那 [[Get]] 操作会返回值 undefined：
  - [[Put]]
    * 很复杂
  - Getter和Setter
    * 在 ES5 中可以使用 getter 和 setter 部分改写默认操作， 但是只能应用在单个属性上
    * 当你给一个属性定义 getter、 setter 或者两者都有时， 这个属性会被定义为“ 访问描述符”。对于访问描述符来说， JavaScript 会忽略它们的 value 和writable 特性， 取而代之的是关心 set 和 get（ 还有 configurable 和enumerable） 特性。
    * 用法
      - 对象文字语法中的 get a() { .. }，
      - defineProperty(..) 中的显式定义
      - 
        ```js
        var myObject = {
          // 给 a 定义一个 getter

          get a() {
            return 2;
          }
        };
        Object.defineProperty(myObject, // 目标对象

          "b", // 属性名

          {
          // 描述符
          // 给 b 设置一个 getter

          get: function(){ return this.a * 2 },
          // 确保 b 会出现在对象的属性列表中

          enumerable: true
          }
        );
          myObject.a; // 2
          myObject.b; // 4
        ```
      - 通常来说 getter 和 setter 是成对出现的（ 只定义一个的话通常会产生意料之外的行为
        * 由于我们只定义了 a 的 getter， 所以对 a 的值进行设置时 set 操作会忽略赋值操作， 不会抛出错误。
        ```js
        var myObject = {
          get a() {
            return 2;
          }
        };
        myObject.a = 3;
        myObject.a; // 2
        ```
  - 存在性
    * in 操作符
      - 会检查属性是否在对象及其 [[Prototype]] 原型链中
      - in 操作符实际上检查的是某个属性名是否存在。 对于数组来说这个区别非常重要， 4 in [2, 4, 6] 的结果并不是你期待的 True， 因为 [2, 4, 6] 这个数组中包含的属性名是 0、 1、2， 没有 4。
      - 在数组上应用 for..in 循环有时会产生出人意料的结果， 因为这种枚举不仅会包含所有数值索引， 还会包含所有可枚举属性。 最好只在对象上应用for..in 循环， 如果要遍历数组就使用传统的 for 循环来遍历数值索引。
    * hasOwnProperty(..) 
      - 只会检查属性是否在 myObject 对象中， 不会检查 [[Prototype]] 链
    * 枚举
      - propertyIsEnumerable(..) 会检查给定的属性名是否直接存在于对象中（ 而不是在原型链上） 并且满足 enumerable:true。
      - Object.keys(..) 会返回一个数组， 包含所有可枚举属性
      - Object.getOwnPropertyNames(..)会返回一个数组， 包含所有属性， 无论它们是否可枚举。
* 遍历
  - 对象
    * for..in 
      - 循环可以用来遍历对象的可枚举属性列表（ 包括 [[Prototype]] 链）
    * for..of
      - 直接遍历值而不是数组下标（ 或者对象属性)
  - 数组
    * for(..)、
    * forEach(..)
    * every(..) 和 some(..)
      - 会提前终止遍历
    * for..of
      - 直接遍历值而不是数组下标（ 或者对象属性)


## 混合对象类
### 类理论
* 继承、实例化和多态
* 类是一种设计模式
* “类”设计模式
  - 迭代器模式、 观察者模式、 工厂模式、 单例模式等
* 类的机制
  - 一个类就是一张蓝图。 为了获得真正可以交互的对象， 我们必须按照类来建造（ 也可以说实例化） 一个东西， 这个东西通常被称为实例
  - 构造函数
    * 类实例是由一个特殊的类方法构造的， 这个方法名通常和类名相同， 被称为构造函数
    * 执行 new 时实际上调用的就是构造函数。
* 类的继承
  - 先定义一个类， 然后定义一个继承前者的类
  - 子类会包含父类行为的原始副本， 但是也可以重写所有继承的行为甚至定义新行为
  - 多态
    * 任何方法都可以引用继承层次中高层的方法


## 原型
### [[Prototype]]
* 几乎所有的对象在创建时都有 [[Prototype]] 属性，指向一个对象
* 对于默认的 [[Get]] 操作来说， 如果无法在对象本身找到需要的属性， 就会继续访问对象的 [[Prototype]] 链
* 使用 for..in 遍历对象时原理和查找 [[Prototype]] 链类似， 任何可以通过原型链访问到（ 并且是 enumerable） 的属性都会被枚举。 使用 in 操作符来检查属性在对象中是否存在时， 同样会查找对象的整条原型链（ 无论属性是否可枚举）
* 所有普通的 [[Prototype]] 链最终都会指向内置的 Object.prototype

* constructor 是一个非常不可靠并且不安全的引用。 通常来说要尽量避免使用这些引用。

*  ES6 添加了辅助函数 Object.setPrototypeOf(..)， 可以用标准并且可靠的方法来修改关联

* instanceof 操作符的左操作数是一个普通的对象， 右操作数是一个函数。 instanceof 回答的问题是： 在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 
这个方法只能处理对象（ a） 和函数（ 带 .prototype 引用的 Foo） 之间的关系

* Foo.prototype.isPrototypeOf( a ); // true
在 a 的整条 [[Prototype]] 链中是否出现过 Foo.prototype

* 我们也可以直接获取一个对象的 [[Prototype]] 链。 在 ES5 中， 标准的方法是：Object.getPrototypeOf( a );