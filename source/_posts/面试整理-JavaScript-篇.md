---
title: 《 JavaScript 高级程序设计》笔记  
date: 2018-01-23 16:58:39
tags:
---

## 标识符
* 定义：指变量、函数、属性的名字，或者函数的参数
* 规则
  - 第一个字符必须是一个字母、下划线（_）或一个美元符号（$）；
  - 其他字符可以是字母、下划线、美元符号或数字。
* 不能把关键字、保留字、 true、 false 和 null 用作标识符。 

## 严格模式

* 严格模式是为 JavaScript 定义了一种不同的
解析与执行模型
* 在顶部或者函数内部的上方 "use strict";

## 关键字和保留字

* 关键字
  - 定义：可用于表示控制语句的开始或结束，或者用于执行特定操作字符
* 保留字
  - 定义：有可能在将来被用作关键字的字符
* 不要使用关键字和保留字作为标识符和属性名

## JS 中基本的数据类型
最新的 ECMAScript 标准定义了 7 种数据类型：
- 6 种 原始类型（也称为基本数据类型）:
  * Boolean
  * Null
  * Undefined
  * Number
  * String
  * Symbol (ECMAScript 6 新定义)
- Object（复杂数据类型)

## typeof 操作符

* typeof 是一个操作符而不是函数，因此例子中的圆括号尽管可以使用，但不是必需的
* 对一个值使用 typeof 操作符可能返回下列某个字符串：
  * "undefined"——如果这个值未定义；
  * "boolean"——如果这个值是布尔值；
  * "string"——如果这个值是字符串；
  * "number"——如果这个值是数值；
  * "object"——如果这个值是对象或 null；
  * "function"——如果这个值是函数
* 对未初始化（即未赋值）的变量执行 typeof 操作符会返回 undefined 值，而对未声明的变量执行 typeof 操作符同样也会返回 undefined 值。
  ```js
  var message; // 这个变量声明之后默认取得了 undefined 值
  // 下面这个变量并没有声明
  // var age
  alert(typeof message); // "undefined"
  alert(typeof age); // "undefined"
  ```

## 关于 null 类型的注意点

* 从逻辑角度来看， null 值表示一个空对象指针，而这也正是使用 typeof 操作符检测 null 值时会返回"object"的原因
* undefined 值是派生自 null 值的，因此 null == undefined // true

## Boolean 类型
* 转换为 false 的值：false, ''（空字符串），0，NaN，null，undefined
* 注意 负数其实是转换成 true 的

## Number 类型
* 八进制
  - 第一位必须是零（0） ，然后是八进制数字序列（0～7） 。
  - 如果字面值中的数值超出了范围，那么前导零将被忽略，后面的数值将被当作十进制数值解析
* 十六进制
  - 前两位必须是 0x，后跟任何十六进制数字（0～9 及 A～F）。其中，字母 A～F 可以大写，也可以小写。
* 浮点数值计算会产生舍入误差
* 数值范围
  - ECMAScript 能够表示的最小数值保存在 Number.MIN_VALUE
  中；能够表示的最大数值保存在 Number.MAX_VALUE 如果某次计算的结果得到了一个超出 JavaScript 数值范围的值，那么这个数值将被自动转换成特殊的 Infinity 值。具体来说，如果这个数值是负数，则会被转换成 -Infinity（负无穷），如果这个数值是正数，则会被转换成 Infinity（正无穷）。
  - isFinite
* NaN
  - 任何涉及 NaN 的操作（例如 NaN/10）都会返回 NaN，
  - NaN 与任何值都不相等，包括 NaN 本身。
  - isNaN 来判断
    * isNaN() 在接收到一个值之后，会尝试
  将这个值转换为数值。某些不是数值的值会直接转换为数值，例如字符串"10"或 Boolean 值。而任何
  不能被转换为数值的值都会导致这个函数返回 true

## String 类型
* 数值、布尔值、对象和字符串值都有 toString() 方法。但 null 和 undefined 值没有这个方法。
* 在调用数值的 toString() 方法时，可
以传递一个参数：输出数值的基数

## 操作符（很多细节）
* 在应用于对象时，相应的操作符通常都会先后调用对象的 valueOf() 和（或） toString() 方法，以便取得可以操作的值
* 位操作符（未看完）
  - 按内存中表示数值的位来操作数值
  - 

## label 语句

* break 和 continue 语句都可以与 label 语句联合使用，从而返回代码中特定的位置
```js
var num = 0;
outermost:
for (var i=0; i < 10; i++) {
  for (var j=0; j < 10; j++) {
    if (i == 5 && j == 5) {
      break outermost;
    }
    num++;
  }
}
alert(num); //55
```

## with 语句
* with 语句的作用是将代码的作用域设置到一个特定的对象中。 
* 
  ```js
  var qs = location.search.substring(1);
  var hostName = location.hostname;
  var url = location.href;
  // 上面几行代码都包含 location 对象。如果使用 with 语句，可以把上面的代码改写成如下所示：
  with(location) {
    var qs = search.substring(1);
    var hostName = hostname;
    var url = href;
  }
  ```
* 在这个重写后的例子中，使用 with 语句关联了 location 对象。这意味着在 with 语句的代码块
内部，每个变量首先被认为是一个局部变量，而如果在局部环境中找不到该变量的定义，就会查询
location 对象中是否有同名的属性。如果发现了同名属性， 则以 location 对象属性的值作为变量的值

## 函数

* ECMAScript 函数不介意传递进来多少个参数，也不在乎传进来参数是什么数据类型
* 在函数体内可以通过 arguments 对象来访问这个参数数组，从而获取传递给函数的每一个参数
  - arguments.length 可以获得实参个数
  - 它的值永远与对应命名参数的值保持同步。
  - 没有传递值的命名参数将自动被赋予 undefined 值。
* 签名：接受的参数的类型和数量

## 基本类型和引用类型的值

* ECMAScript 变量可能包含两种不同数据类型的值：基本类型值和引用类型值。 基本类型值指的是简单的数据段，而引用类型值指那些可能由多个值构成的对象。
* 基本数据类型： Undefined、 Null、 Boolean、 Number 和 String。这 5 种基本数据类型是按值访问的，因为可以操作保存在变量中的实际的值。保存在栈中
* 引用类型的值是保存在内存中的对象。在操作对象时，实际上是在操作对象的引用而不是实际的对象。保存在堆中
* 如果从一个变量向另一个变量复制基本类型的值，会在变量对象上创建一个新值，然后把该值复制
到为新变量分配的位置上。当从一个变量向另一个变量复制引用类型的值时，同样也会将存储在变量对象中的值复制一份放到为新变量分配的空间中。不同的是，这个值的副本实际上是一个指针，而这个指针指向存储在堆中的一个对象。复制操作结束后，两个变量实际上将引用同一个对象。因此，改变其中一个变量，就会影响另一个变量
* 传递参数
  - ECMAScript 中所有函数的参数都是按值传递的。也就是说，把函数外部的值复制给函数内部的参数，就和把值从一个变量复制到另一个变量一样。基本类型值的传递如同基本类型变量的复制一样，而引用类型值的传递，则如同引用类型变量的复制一样。
  - 在向参数传递基本类型的值时，被传递的值会被复制给一个局部变量（即命名参数，或者用
  ECMAScript 的概念来说，就是 arguments 对象中的一个元素）。在向参数传递引用类型的值时，会把这个值在内存中的地址复制给一个局部变量，因此这个局部变量的变化会反映在函数的外部
  - 
  ```js
  function addTen(num) {
    num += 10;
    return num;
  }
  var count = 20;
  var result = addTen(count);
  alert(count); //20，没有变化
  alert(result); //30
  // 个人理解，可以改写成
  function addTen(num) {
    // 赋值成局部变量
    // 以把 ECMAScript 函数的参数想象成局部变量。
    var num = num
    num += 10;
    return num;
  }
  ```

## 检测类型
* 检测一个变量是不是基本数据类型
  * typeof 检测基本数据类型，不能检查不 null
  * instanceof 检查引用类型

## 执行环境及作用域
* 执行环境（也称为作用域）的类型总共只有两种：全局和局部（函数）
* 标识符解析是沿着作用域链一级一级地搜索标识符的过程。搜索过程始终从作用域链的前端开始，然后逐级地向后回溯，直至找到标识符为止（如果找不到标识符，通常会导致错误发生）。如果在局部环境中找到了该标识符，搜索过程停止，变量就绪。如果在局部环境中没有找到该变量名，则继续沿作用域链向上搜索。搜索过程将一直追溯到全局环境的变量对象。如果在全局环境中也没有找到这个标识符，则意味着该变量尚未声明。
* 内部环境可以通过作用域链访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数。这些环境之间的联系是线性、有次序的。每个环境都
可以向上搜索作用域链，以查询变量和函数名；但任何环境都不能通过向下搜索作用域链而进入另一个
执行环境。
* 访问局部变量要比访问全局变量更快，因
为不用向上搜索作用域链
* 延长作用域链
  * with 语句
  * try...catch

## 垃圾收集

* 局部变量只在函数执行的过程中存在。然后在函数中使用这些变量，直至函数执行结束。此时，局部变量就没有存在的必要了，因此可以释放它们的内存以供将来使用。垃圾收集器必须跟踪哪个变量有用哪个变量没用，对于不再有用的变量打上标记，以备将来收回其占用的内存。
* 分类
  * 标记清除
    - 给当前不使用的值加上标记，然后再回收其内存
  * 引用计数（不常用）
    - 循环引用
* 一旦数据不再有用，最好通过将其值设置为 null 来释放其引用——这个做法叫做解除引用（dereferencing）。

## 引用类型
* Object, Array

## Object 类型

* 创建 Object 实例的方式有两种：使用 new 操作符后跟 Object 构造函数 , 对象字面量
* 在通过对象字面量定义对象时，实际上不会调用 Object 构造函数

## Array 类型

* 数组的每一项可以保存任何类型的数据
* 与对象一样，在使用数组字面量表示法时，也不会调用 Array 构造函数
* 检测数组：Array.isArray
* 调用数组的 toString() 方法会返回由数组中每个值的字符串形式拼接而成的一个以逗号分隔的字符串。
* alert() 要接收字符串参数，所以它会在后台调用 toString() 方法，由此会得到与直接调用 toString() 方法相同的结果。
* 如果数组中的某一项的值是 null 或者 undefined，那么该值在 join()、toLocaleString()、 toString() 和 valueOf() 方法返回的结果中以空字符串表示。
* 数组可以用来模拟栈和队列
  - 栈 (LIFO)：后进先出
    * push: 返回修改后数组的长度
    * pop: 返回移除的项
  - 队列 (FIFO)：先进先出
    * push: 返回修改后数组的长度
    * shift: 返回移除的项
    * unshift: 返回修改后数组的长度
* 重排序方法
  * reverse
  * sort 方法：
    - 在默认情况下， sort() 方法按升序排列数组项——即最小的值位于最前面，最大的值排在最后面。为了实现排序， sort() 方法会调用每个数组项的 toString() 转型方法，然后比较得到的字符串，以确定如何排序。即使数组中的每一项都是数值， sort() 方法比较的也是字符串
    - sort() 方法可以接收一个比较函数作为参数。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个之后则返回一个正数。
    - 
      ```js
        function compare(value1, value2){
          return value2 - value1;
        }
      ```
* splice 方法：
  - 删除：可以删除任意数量的项，只需指定 2 个参数：要删除的第一项的位置和要删除的项数。例如， splice(0,2) 会删除数组中的前两项。
  - 插入：可以向指定位置插入任意数量的项，只需提供 3 个参数：起始位置、 0（要删除的项数）和要插入的项。如果要插入多个项，可以再传入第四、第五，以至任意多个项。例如，splice(2,0,"red","green") 会从当前数组的位置 2 开始插入字符串"red"和"green"。
  - 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定 3 个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice (2,1,"red","green") 会删除当前数组位置 2 的项，然后再从位置 2 开始插入字符串"red"和"green"。

## (Date 和正则没有看 )

## Function 

* 函数实际上是对象。每个函数都是 Function 类型的实例，而且都与其他引用类型一样具有属性和方法
* 由于函数是对象，因此函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定
* 不推荐使用 Function 构造函数定义函数，因为这种语法会导致解析两次代码（第一次是解析常规 ECMAScript 代码，第二次是解析传入构造函数中的字符串），从而影响性能。
* 解析器会率先读取函数声明，并使其在执行任何代码之前可用（可以访问）；至于函数表达式，则必须等到解析器执行到它所在的代码行，才会真正被解释执行
*  arguments 
  - 主要用途是保存函数参数，但这个对象还有一个名叫 callee 的属性，该属性是一个指针，指向拥有这个 arguments 对象的函数。
  - caller 保存着调用当前函数的函数的引用
* 每个函数都包含两个属性：length 和 prototype。
  - length 属性表示函数形参的个数
  - 对于引用类型而言， prototype 是保存它们所有实例方法的真正所在。换句话说，诸如 toString() 和 valueOf() 等方法实际上都保存在 prototype 名下，只不过是通过各自对象的实例访问。在 ECMAScript 5 中， prototype 属性是不可枚举的，因此使用 for-in 无法发现

## 基本包装类型

* ECMAScript 提供了 3 个特殊的引用类型： Boolean、 Number 和 String。
* 实际上，每当读取一个基本类型值的时候，后台就会创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。
```js
var s1 = "some text";
var s2 = s1.substring(2);
```
这个例子中的变量 s1 包含一个字符串，字符串当然是基本类型值。而下一行调用了 s1 的 substring() 方法，并将返回的结果保存在了 s2 中。我们知道，基本类型值不是对象，因而从逻辑上讲它们不应该有方法（尽管如我们所愿，它们确实有方法）。

其实，为了让我们实现这种直观的操作，后台已经自动完成了一系列的处理。当第二行代码访问 s1 时，访问过程处于一种读取模式，也就是要从内存中读取这个字符串的值。而在读取模式中访问字符串时，后台都会自动完成下列处理
(1) 创建 String 类型的一个实例；
(2) 在实例上调用指定的方法；
(3) 销毁这个实例。
可以将以上三个步骤想象成是执行了下列 ECMAScript 代码。
```js
var s1 = new String("some text");
var s2 = s1.substring(2);
s1 = null;
```
经过此番处理，基本的字符串值就变得跟对象一样了。而且，上面这三个步骤也分别适用于 Boolean 和 Number 类型对应的布尔值和数字值。
* 自动创建的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。这意味着我们不能在运行时为基本类型值添加属性和方法。

* Number 类型
  - toString 方法：传递一个表示基数的参数，告诉它返回几进制字符串格式
  - toFixed: 按照指定的小数位返回数值的字符串。会有自动舍入
  - toExponential：以指数表示法（也称 e 表示法）表示的数值的字符串形式

* String 类型
  - charAt() 方法：返回给定位置的那个字符
  - charCodeAt()：返回指定位置字字符编码
  - 使用方括号加数字索引来访问字符串中的特定字符
  - concat()： 用于将一或多个字符串拼接起来，返回拼接得到的新字符串
  - slice
  - substring：基本用法和 slice 一样
    * substring() 方法会把所有负值参数都转换为 0。
  - substr：第二个参数指定的则是返回的字符个数
    * substr() 方法将负的第一个参数加上字符串的长度，而将负的第二个参数转换为 0。
  - trim
  - toLowerCase()、 toLocaleLowerCase()、 toUpperCase() 和 toLocaleUpperCase()
  - 字符串的模式匹配方法（没看）
    * match：只接受一个参数，要么是一个正则表达式，要么是一个 RegExp 对象，返回一个数组
    * search: 接受一个正则，返回字符串中第一个匹配项的索引
    * replace
      - 第一个参数可以是一个 RegExp 对象或者一个字符串，第二个参数可以是一个字符串或者一个函数。

## Global 对象

* 不属于任何其他对象的属性和方法，最终都是它的属性和方法
* 诸如 isNaN()、isFinite()、parseInt() 以及 parseFloat()，实际上全都是 Global 对象的方法
* URI 编码方法
  - encodeURI：不会对本身属于 URI 的特殊字符进行编码，例如冒号、正斜杠、问号和井字号
  - encodeURIComponent()：会对它发现的任何非标准字符进行编码。
* eval
  - 只接受一个参数，即要执行的 ECMAScript （或 JavaScript）字符串。
  ```js
    eval("alert('hi')");
    // 这行代码的作用等价于下面这行代码：
    alert("hi");
  ```
  - 在 eval() 中创建的任何变量或函数都不会被提升，因为在解析代码的时候，它们被包含在一个字符串中；它们只在 eval() 执行的时候创建

## Math 对象
* min() 和 max() 方法用于确定一组数值中的最小值和最大值。这两个方法都可以接收任意多个数值参数，
* 舍入方法
  - Math.ceil() 执行向上舍入，即它总是将数值向上舍入为最接近的整数；
  - Math.floor() 执行向下舍入，即它总是将数值向下舍入为最接近的整数；
  - Math.round() 执行标准舍入，即它总是将数值四舍五入为最接近的整数
* Math.random() 方法返回大于等于 0 小于 1 的一个随机数

## 面向对象

* ECMAScript 中没有类的概念，因此它的对象也与基于类的语言中的对象有所不同
* 我们可以把 ECMAScript 的对象想象成散列表：就是一组名值对，其中值可以是数据或函数
* 对象的属性类型
  - 定义这些特性是为了实现 JavaScript 引擎用的，因此在 JavaScript 中不能直接访问它们。为了表示特性是内部值，该规范把它们放在了两对儿方括号中，例如 [[Enumerable]]
  - ECMAScript 中有两种属性：数据属性和访问器属性
  - 数据属性
    * [[Configurable]]：是否可删除、是否可配置
    * [[Enumerable]]：能否通过 for-in 循环返回遍历
    * [[Writable]]：能否修改属性的值
    * [[Value]]：这个属性的数据值
  - 要修改属性默认的特性，必须使用 ECMAScript 5 的 Object.defineProperty() 方法
    * Object.defineProperty()
      - 修改属性默认的特性
      - 接收三个参数：属性所在的对象、属性的名字和一个描述符对象
      - 可以多次调用 Object.defineProperty() 方法修改同一个属性，但在把 configurable 特性设置为 false 之后就会有限制了
  - 访问器属性：getter 和 setter
    * [[Configurable]]：是否可删除、是否可配置
    * [[Enumerable]]：能否通过 for-in 循环返回属性
    * [[Get]]：在读取属性时调用的函数
    * [[Set]]：在写入属性时调用的函数
    * 访问器属性必须使用 Object.defineProperty() 来定义
    * 不一定非要同时指定 getter 和 setter
  - Object.defineProperties
    * 这个方法可以通过描述符一次定义多个属性
    * 这个方法接收两个对象参数：第一个对象是要添加和修改其属性的对象，第二个对象的属性与第一个对象中要添加或修改的属性一一对应
  - Object.getOwnPropertyDescriptor
    * 取得给定属性的描述符
    * 接收两个参数：属性所在的对象和要读取其描述符的属性名称
* 创建对象
  - 工厂模式
    ```js
      function createPerson(name, age, job) {
        var o = new Object();
        o.name = name;
        o.age = age;
        o.job = job;
        o.sayName = function() {
          alert(this.name);
        };
        return o;
      }
      var person1 = createPerson("Nicholas", 29, "Software Engineer");
    ```
  - 构造函数模式
    * 
      ```js
        function Person(name, age, job){
          this.name = name;
          this.age = age;
          this.job = job;
          this.sayName = function(){
            alert(this.name);
          };
        }
        var person1 = new Person("Nicholas", 29, "Software Engineer");
      ```
    * 以这种方式调用构造函数实际上会经历以下 4 个步骤：
      1. 创建一个新对象；
      2. 将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）；
      3. 执行构造函数中的代码（为这个新对象添加属性）；
      4. 返回新对象。
    * 构造函数也是普通函数，任何函数只要通过 new 操作符来调用，那它就可以作为构造函数
    * 使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍

  - 原型模式
    * 每个函数都有一个 prototype（原型）属性，这个属性是一个指针，指向一个对象，包含所有实例共享的属性和方法。所有原型对象都会自动获得一个 constructor（构造函数）属性，这个属性指向 prototype 属性所在函数
    * 实例自身有一个 [[Prototype]]，指向构造函数的 prototype （原型对象）。可以通过 __proto__ 访问
    * isPrototypeOf，Object.getPrototypeOf 可以用来判断原型对象
    * 每当代码读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。搜索首先从对象实例本身开始。如果在实例中找到了具有给定名字的属性，则返回该属性的值；如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性。如果在原型对象中找到了这个属性，则返回该属性的值。
    * 虽然可以通过对象实例访问保存在原型中的值，但却不能通过对象实例重写原型中的值。如果我们在实例中添加了一个属性，而该属性与实例原型中的一个属性同名，那我们就在实例中创建该属性，该属性将会屏蔽原型中的那个属性。
    * 无论该属性存在于实例中还是存在于原型中，调用 in 始终都返回 true， 使用 hasOwnProperty() 方法可以检测一个属性是存在于实例中，还是存在于原型中。
    * 在使用 for-in 循环时，返回的是所有能够通过对象访问的、可枚举的（enumerated）属性，其中既包括存在于实例中的属性，也包括存在于原型中的属性。
    * IE 早期版本的实现中存在一个 bug，即屏蔽不可枚举属性的实例属性不会出现在 for-in 循环中
      - 
      ```js
      var o = {
        toString : function(){
          return "My Object";
        }
      };
      for (var prop in o){
        if (prop == "toString"){
          alert("Found toString"); // 在 IE 中不会显示
        }
      }
      ```
    * 要取得对象上所有可枚举的实例属性，不包含原型上的，可以使用 Object.keys() 方法
    * 如果你想要得到所有实例属性，无论它是否可枚举，都可以使用 Object.getOwnPropertyNames() 方法。

  - 组合使用构造函数模式和原型模式
    * 实例属性都是在构造函数中定义的，而由所有实例共享的属性 constructor 和方法则是在原型中定义的
  - 动态原型模式
  - 寄生构造函数模式
    * 这种模式的基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数
    ```js
      function Person(name, age, job){
        var o = new Object();
        o.name = name;
        o.age = age;
        o.job = job;
        o.sayName = function(){
          alert(this.name);
        };
        return o;
      }
      var friend = new Person("Nicholas", 29, "Software Engineer");
    ```
* 继承
  - 原型链
    * 基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法
    * 实现的本质是重写原型对象，代之以一个新类型的实例。
    * 所有函数的默认原型都是 Object 的实例，因此默认原型都会包含一个内部指针，指向 Object.prototype。
    * 确定原型和实例的关系
      - instanceof 操作符
      - isPrototypeOf() 方法
  - 借用构造函数
    * 即在子类型构造函数的内部调用超类型构造函数。
      ```js
        function SuperType(){
          this.colors = ["red", "blue", "green"];
        }
        function SubType(){
          // 继承了 SuperType
          SuperType.call(this);
        }
        var instance1 = new SubType();
        instance1.colors.push("black");
        alert(instance1.colors); //"red,blue,green,black"
        var instance2 = new SubType();
        alert(instance2.colors); //"red,blue,green"
      ```
  - 组合继承
    * 指的是将原型链和借用构造函数的技术组合到一块。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承
    ```js
      function SuperType(name){
        this.name = name;
        this.colors = ["red", "blue", "green"];
      }
      SuperType.prototype.sayName = function(){
        alert(this.name);
      };
      function SubType(name, age){
        // 继承属性
        SuperType.call(this, name);
        this.age = age;
      }
      // 继承方法
      SubType.prototype = new SuperType();
      SubType.prototype.constructor = SubType;
      SubType.prototype.sayAge = function(){
        alert(this.age);
      };
    ```
  - 原型式继承
    * 从本质上讲， object() 对传入其中的对象执行了一次浅复制
    ```js
      function object(o){
        function F(){}
        F.prototype = o;
        return new F();
      }
    ```
    * Object.create() 方法规范化了原型式继承
  - 寄生式继承
    ```js
      function createAnother(original){
        var clone = object(original); // 通过调用函数创建一个新对象
        clone.sayHi = function(){ // 以某种方式来增强这个对象
        alert("hi");
      };
        return clone; // 返回这个对象
      }
    ```
  - 寄生组合式继承
    ```js
      function inheritPrototype(subType, superType){
        var prototype = object(superType.prototype); // 创建对象
        prototype.constructor = subType; // 增强对象
        subType.prototype = prototype; // 指定对象
      }
    ```
    * 这个例子的高效率体现在它只调用了一次 SuperType 构造函数，并且因此避免了在 SubType.prototype 上面创建不必要的、多余的属性

## 函数表达式
### 
* 通过 name 属性可以访问到给函数指定的名字
* 函数声明提升：可以把函数声明放在调用它的语句后面。
* 匿名函数
* 递归
  - 递归函数是在一个函数通过名字调用自身的情况下构成的
  - arguments.callee 是一个指向正在执行的函数的指针，因此可以用它来实现对函数的递归调用
* 闭包
  - 闭包是指有权访问另一个函数作用域中的变量的函数
  - 在函数执行过程中，为读取和写入变量的值，就需要在作用域链中查找变量。
  - 当某个函数被调用时，会创建一个执行环境（execution context）及相应的作用域链。然后，使用 arguments 和其他命名参数的值来初始化函数的活动对象（activation object）。但在作用域链中，外部函数的活动对象始终处于第二位，外部函数的外部函数的活动对象处于第三位，……直至作为作用域链终点的全局执行环境。在函数执行过程中，为读取和写入变量的值，就需要在作用域链中查找变量。
  - 我们知道， this 对象是在运行时基于函数的执行环境绑定的：在全局函数中， this 等于 window，而当函数被作为某个对象的方法调用时， this 等
于那个对象
  - http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html
  - 内存泄漏
    * IE9 之前如果闭包的作用域链中保存着一个 HTML 元素，那么就意味着该元素将无法被销毁
  - 模仿块级作用域
    * 匿名函数可以用来模仿块级作用域
    ```js
    (function(){
      // 这里是块级作用域
    })();
    ```
    * 这种技术经常在全局作用域中被用在函数外部，从而限制向全局作用域中添加过多的变量和函数。一般来说，我们都应该尽量少向全局作用域中添加变量和函数
  - 私有变量
    * 任何在函数中定义的变量，都可以认为是私有变量
    * 利用私有和特权成员，可以隐藏那些不应该被直接修改的数据，```js
      function Person(name){
        this.getName = function(){
          return name;
        };
        this.setName = function (value) {
          name = value;
        };
      }
      var person = new Person("Nicholas");
      alert(person.getName()); //"Nicholas"
      person.setName("Greg");
      alert(person.getName()); //"Greg"
    ```
  - 模块模式（单例模式）
    * 指的就是只有一个实例的对象
    * 模块模式通过为单例添加私有变量和特权方法能够使其得到增强
    ```js
    var singleton = function(){
      // 私有变量和私有函数
      var privateVariable = 10;
      function privateFunction(){
        return false;
      }
      // 特权 / 公有方法和属性
      return {
        publicProperty: true,
        publicMethod : function(){
          privateVariable++;
          return privateFunction();
        }
      };
    }();
    ```

## BOM（浏览器对象模型)

### window 对象
* 全局作用域
  - 由于 window 对象同时扮演着 ECMAScript 中 Global 对象的角色，因此所有在全局作用域中声明的变量、函数都会变成 window 对象的属性和方法
  - 全局变量不能通过 delete 操作符删除，而直接在 window 对象上的定义的属性可以
  ```js
  var age = 29;
  window.color = "red";
  // 在 IE < 9 时抛出错误，在其他所有浏览器中都返回 false
  delete window.age;
  // 在 IE < 9 时抛出错误，在其他所有浏览器中都返回 true
  delete window.color; //returns true
  alert(window.age); //29
  alert(window.color); //undefined
  ```
  - var 语句添加的 window 属性有一个名为 [[Configurable]] 的特性，这个特性的值被设置为 false，因此这样定义的属性不可以通过 delete 操作符删除
  - 
* 窗口关系及框架
  - 如果页面中包含框架 (frame)，则每个框架都拥有自己的 window 对象，并且保存在 frames 集合中，每个 window 对象都有一个 name 属性，其中包含框架的名称
* 窗口位置
  - 
* 窗口大小
  - 


## XMLHttpRequest

## 兼容
* IE10/IE11 部分支持
* new ActiveXObject("Microsoft.XMLHTTP")

### XHR 的用法
* 创建 xhr 对象 var xhr = new XMLHttpRequest();
* xhr.open("get", "example.php", false);
  - 接受 3 个参数：要发送的请求的类型（"get"、 "post"等） 、请求的 URL 和表示是否异步发送请求的布尔值
  - URL 相对于执行代码的当前页面（当然也可以使用绝对路径）
  - 调用 open() 方法并不会真正发送请求，而只是启动一个请求以备发送
* xhr.send(null);
  - 这里的 send() 方法接收一个参数，即要作为请求主体发送的数据。如果不需要通过请求主体发送数据，则必须传入 null
* xhr.setRequestHeader
  - 可以设置自定义的请求头部信息
* 响应
  - readyState
    * 表示请求 / 响应过程的当前活动阶段
    * 0：未初始化。尚未调用 open() 方法。
    * 1：启动。已经调用 open() 方法，但尚未调用 send() 方法。
    * 2：发送。已经调用 send() 方法，但尚未接收到响应。
    * 3：接收。已经接收到部分响应数据。
    * 4：完成。已经接收到全部响应数据，而且已经可以在客户端使用了。
    * 只要 readyState 属性的值由一个值变成另一个值，都会触发一次 readystatechange 事件。
  - responseText：作为响应主体被返回的文本。
  - responseXML：如果响应的内容类型是"text/xml"或"application/xml"，这个属性中将保存包含着响应数据的 XML DOM 文档。
  - status：响应的 HTTP 状态。
  - statusText： HTTP 状态的说明

### GET 请求
* 查询字符串中每个参数的名称和值都必须使用 encodeURIComponent() 进行编码，然后才能放到 URL 的末尾

### POST 请求
* 使用 XHR 来模仿表单提交：首先将 Content-Type 头部信息设置为 application/x-www-form-urlencoded，

## XMLHttpRequest 2 级
* FormData
  - append() 方法接收两个参数：键和值
  ```js
    var data = new FormData();
    data.append("name", "Nicholas");
    xhr.send(data);
  ```
* 超时设定
  - 表示请求在等待响应多少毫秒之后就终止
  -  在给 timeout 设置一个数值后，如果在规定的时间内浏览器还没有接收到响应，那么就会触发 timeout 事件，进而会调用 ontimeout 事件处理程序
  ```js
  xhr.timeout = 1000; // 将超时设置为 1 秒钟（仅适用于 IE8+）
  xhr.ontimeout = function(){
    alert("Request did not return in a second.");
  };
  ```
* overrideMimeType() 方法
  - 重写 XHR 响应的 MIME 类型
  - 调用 overrideMimeType() 必须在 send() 方法之前，才能保证重写响应的 MIME 类 xhr.overrideMimeType("text/xml");

## 进度事件
* load 事件
  - 响应接收完毕后将触发 load 事件，因此也就没有必要去检查 readyState 属性了
  ```js
  var xhr = createXHR();
  xhr.onload = function(){
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
    alert(xhr.responseText);
  } else {
    alert("Request was unsuccessful: " + xhr.status);
  }
  };
  xhr.open("get", "altevents.php", true);
  xhr.send(null);
  ```
* progress 事件
  - 这个事件会在浏览器接收新数据期间周期性地触发
  - lengthComputable 是一个表示进度信息是否可用的布尔值， position 表示已经接收的字节数， totalSize 表示根据 Content-Length 响应头部确定的预期字节数
  ```js
  xhr.onprogress = function(event){
  var divStatus = document.getElementById("status");
  if (event.lengthComputable){
    divStatus.innerHTML = "Received " + event.position + " of " +
    event.totalSize +" bytes";
  }
  };
  ```

## 长轮询是传统轮询（也称为短轮询）的一个翻版，即浏览器定时向服务器发送请求，看有没有更新的数据

## Web Sockets
* Web Sockets API
  - var socket = new WebSocket("ws://www.example.com/server.php");
  必须给 WebSocket 构造函数传入绝对 URL。
* 发送和接收数据
  - socket.send("Hello world!")
  - Web Sockets 只能通过连接发送纯文本数据，所以对于复杂的数据结构，在通过连接发送之前，必须进行序列化
  - 当服务器向客户端发来消息时， WebSocket 对象就会触发 message 事件
* 其他事件
  - open，error，close
  - WebSocket 对象不支持 DOM 2 级事件侦听器，因此必须使用 DOM 0 级语法分别定义每个事件处（onopen）

## DOM（文档对象模型)
### 概述
  * DOM 描绘了一个层次化的节点树，允许开发人员添加、移除和修改页面的某一部分
### 节点层次
  * 文档节点是每个文档的根节点，即<html>元素，称之为文档元素
  * Node 类型
    - 每个节点都有一个 nodeType 属性，用于表明节点的类型
    - 节点类型由在 Node 类型中定义的 12 个数值常量来表示，任何节点类型必居其一
      * Node.ELEMENT_NODE(1)，Node.TEXT_NODE(3)
      * 为了确保跨浏览器兼容，最好还是将 nodeType 属性与数字值进行比较
    - nodeName: 元素的标签名，大写字符串( ex: DIV )
      * 对于元素节点，nodeName 中保存的始终都是元素的标签名，而 nodeValue 的值则始终为 null
    - 节点关系
      * 每个节点都有一个 childNodes 属性，其中保存着一个 NodeList 对象，NodeList 是一种类数组对象
        - 用于保存一组有序的节点，可以通过位置来访问这些节点
        - DOM 结构的变化能够自动反映在 NodeList 对象中
        - 将 NodeList 对象转换为数组：Array.prototype.slice.call(someNode.childNodes,0)
      * hasChildNodes 在节点包含一或多个子节点的情况下返回 true
      * 每个节点都有一个 parentNode 属性，该属性指向文档树中的父节点
      * previousSibling 和 nextSibling 属性，可以访问同一列表中的其他兄弟节点
        - 列表中第一个节点的 previousSibling 属性值为 null，而列表中最后一个节点的 nextSibling 属性的值同样也为 null
        - 如果列表中只有一个节点，那么该节点的 nextSibling 和 previousSibling 都为 null
      * 父节点的 firstChild 和 lastChild 属性分别指向其 childNodes 列表中的第一个和最后一个节点
    - 操作节点
      * appendChild
        - 向 childNodes 列表的末尾添加一个节点
        - 更新完成后，appendChild() 返回新增的节点
        - 如果传入到 appendChild()中的节点已经是文档的一部分了，那结果就是将该节点从原来的位置转移到新位置(任何 DOM 节点不能同时出现在文档中的多个位置上)
      * insertBefore
        - 把节点放在 childNodes 列表中某个特定的位置上
        - 接受两个参数：要插入的节点和作为参照的节点。插入节点后，被插入的节点会变成参照节点的前一个同胞节点, insertBefore() 返回被插入的节点
        - 如果参照节点是 null，则 insertBefore()与 appendChild()执行相同的操作，
      * replaceChild
        - 替换一个节点
        - 接受的两个参数是：要插入的节点和要替换的节点
        - 要替换的节点将由这个方法返回并从文档树中被移除，同时由要插入的节点占据其位置
      * removeChild
        - 移除一个节点
        - 这个方法接受一个参数，即要移除的节点。被移除的节点将成为方法的返回值
      * cloneNode
        - 所有类型的节点都有, 用于创建调用这个方法的节点的一个完全相同的副本
        - 接受一个布尔值参数，表示是否执行深复制
          * 在参数为 true 的情况下，执行深复制，也就是复制节点及其整个子节点树
          * 在参数为 false 的情况下，执行浅复制，即只复制节点本身
      * normalize
        - 处理文档树中的文本节点
        - 由于解析器的实现或 DOM 操作等原因，可能会出现文本节点不包含文本，或者接连出现两个文本节点的情况。当在某个节点上调用这个方法时，就会在该节点的后代节点中查找上述两种情况。如果找到了空文本节点，则删除它；如果找到相邻的文本节点，则将它们合并为一个文本节点
==================================
* Document 类型
  - nodeType 的值为 9
  - 文档的子节点
    * document.documentElement: 指向 HTML 页面中的<html>元素
    * document.childNodes
    * document.body 直接指向<body>元素
    * document.doctype: 取得对<!DOCTYPE>的引用
  - 文档信息
    * document.title: 包含着<title>元素中的文本，修改 title 属性的值不会改变<title>元素。
    * document.URL: 地址栏中显示的 URL
    * document.domain: 包含页面的域名
      - 可以设置，用来跨域
    * document.referrer: 保存着链接到当前页面的那个页面的 URL
  - 查找元素
    * getElementById
      - 如果不存在带有相应 ID 的元素，则返回 null
      - 如果页面中多个元素的 ID 值相同， getElementById() 只返回文档中第一次出现的元素
    * getElementsByTagName
      - 返回的是包含零或多个元素的 NodeList
      - document.getElementsByTagName("*") 可以取得文档中的所有元素
    * getElementsByName
      - 这个方法会返回带有给定 name 特性的所有元素 
  - 文档写入
    * write()、 writeln()、 open() 和 close()。
* Element 类型
  - nodeType 的值为 1
  - nodeName 的值为元素的标签名
  - 要访问元素的标签名，可以使用 nodeName 属性，也可以使用 tagName 属性。
    * 标签名始终都以全部大写表示
    * if (element.tagName.toLowerCase() == "div"){ } // 这样最好
  - HTML 元素
    * someNode.id //"myDiv""
    * someNode.className //"bd fl"
    * someNode.title //"Body text"
    * someNode.lang //"en"
    * someNode.dir
  - 取得特性
    * getAttribute
      - div.getAttribute("class")
      - 只有在取得自定义特性值的情况下，才会使用 getAttribute
    * setAttribute
      - 方法接受两个参数：要设置的特性名和值
      - div.setAttribute("class", "ft");
      - 如果特性已经存在，setAttribute() 会以指定的值替换现有的值；如果特性不存在，setAttribute()
        则创建该属性并设置相应的值
    * removeAttribute
      - div.removeAttribute("class");
  - attributes 属性
    *
  - 创建元素
    * 使用 document.createElement() 方法可以创建新元素。
    * 这个方法只接受一个参数，即要创建元素的标签名。
  - 元素的子节点
    * childNodes 属性中包含了它的所有子节点，这些子节点有可能是元素、文本节点、注释或处理指令。 
    * 空白符会被解析成文本节点
    * 遍历执行某项操作以前，通常都要先检查一下 nodeTpye 属性
* Text 类型
  - nodeType 的值为 3
  - nodeValue 的值为节点所包含的文本
  - 不支持（没有）子节点






## 不在块中定义函数时，先提升函数，再提升变量声明。

## React 相关

* setState
  - 不会立刻改变 React 组件中 state 的值；
    * render 函数被重新执行时 this.state 才被改变。
  - 多次 setState 函数调用产生的效果会合并。
  - setState 通过引发一次组件的更新过程来引发重新绘制；
  - 同步更新 state 的方法
    * 传递给 this.setState 一个函数
    * 在 React 中，如果是由 React 引发的事件处理（比如通过 onClick 引发的事件处理），调用 setState 不会同步更新 this.state，除此之外的 setState 调用会同步执行 this.state。所谓“除此之外”，指的是绕过 React 通过 addEventListener 直接添加的事件处理函数，还有通过 setTimeout/setInterval 产生的异步调用。

* 每个函数都有一个属性叫做 prototype。
* 这个 prototype 的属性值是一个对象（属性的集合，再次强调！），默认的只有一个叫做 constructor 的属性，指向这个函数本身。
* 每个对象都有一个隐藏的属性——“__proto__”，这个属性引用了创建这个对象的函数的 prototype。即：fn.__proto__ === Fn.prototype。这里的"__proto__"成为“隐式原型”
* Instanceof 运算符的第一个变量是一个对象，暂时称为 A；第二个变量一般是一个函数，暂时称为 B。
Instanceof 的判断队则是：沿着 A 的__proto__这条线来找，同时沿着 B 的 prototype 这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回 true。如果找到终点还未重合，则返回 false。