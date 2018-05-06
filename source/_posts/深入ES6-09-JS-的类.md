---
title: 深入ES6 09-JS 的类
date: 2017-12-12 14:29:19
tags:
---

与大多数正规的面向对象编程语言不同， JS 从创建之初就不支持类，也没有把类继承作为定义相似对象以及关联对象的主要方式，这让不少开发者感到困惑。而从 ES1 诞生之前直到 ES5 时期，很多库都创建了一些工具，使 JS 看起来仿佛能支持类。尽管一些 JS 开发者强烈
认为这门语言不需要类，但为处理类而创建的代码库如此之多，导致 ES6 最终引入了类。

在探索 ES6 的类的过程中，理解类的潜在机制会很有帮助，因此本章将会首先讨论 ES5 的开发者如何实现对类行为的模仿。然而正如你将在后面看到的， ES6 的类并不与其他语言的类完全相同，所具备的独特性正配合了 JS 的动态本质

## ES5 中的仿类结构
JS 在 ES5 及更早版本中都不存在类。与类最接近的是：创建一个构造器，然后将方法指派到该构造器的原型上。这种方式通常被称为创建一个自定义类型。例如：
```js
function PersonType(name) {
  this.name = name;
}
PersonType.prototype.sayName = function() {
  console.log(this.name);
};
let person = new PersonType("Nicholas");
person.sayName(); // 输出 "Nicholas"
console.log(person instanceof PersonType); // true
console.log(person instanceof Object); // true
```

此代码中的 PersonType 是一个构造器函数，并创建了单个属性 name 。 sayName() 方法被指派到原型上，因此在 PersonType 对象的所有实例上都共享了此方法。接下来，使用 new 运算符创建了 PersonType 的一个新实例 person ，此对象能被判定是通过原型而继承了 PersonType 与 Object 的实例。

这种基本模式在许多对类进行模拟的 JS 库中都存在，而这也是 ES6 类的出发点。

## 类的声明

类在 ES6 中最简单的形式就是类声明，它看起来很像其他语言中的类。

### 基本的类声明

类声明以 class 关键字开始，其后是类的名称；剩余部分的语法看起来就像对象字面量中的方法简写，并且在方法之间不需要使用逗号。作为范例，此处有个简单的类声明：
```js
class PersonClass {
  // 等价于 PersonType 构造器
  constructor(name) {
    this.name = name;
  }
  // 等价于 PersonType.prototype.sayName
  sayName() {
    console.log(this.name);
  }
}

let person = new PersonClass("Nicholas");
person.sayName(); // 输出 "Nicholas"
console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true
console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass.prototype.sayName); // "function"
```

这个 PersonClass 类声明的行为非常类似上个例子中的 PersonType 。类声明允许你在其中使用特殊的 constructor 方法名称直接定义一个构造器，而不需要先定义一个函数再把它当作构造器使用。由于类的方法使用了简写语法，于是就不再需要使用 function 关键字。constructor 之外的方法名称则没有特别的含义，因此可以随你高兴自由添加方法。

自有属性（Own properties ） ：该属性出现在实例上而不是原型上，只能在类的构造器或方法内部进行创建。在本例中， name 就是一个自有属性。我建议应在构造器函数内创建所有可能出现的自有属性，这样在类中声明变量就会被限制在单一位置（ 有助于代码检查） 。

有趣的是，相对于已有的自定义类型声明方式来说，类声明仅仅是以它为基础的一个语法糖。 PersonClass 声明实际上创建了一个拥有 constructor 方法及其行为的函数，这也是 typeof PersonClass 会得到 "function" 结果的原因。此例中的 sayName() 方法最终也成为 PersonClass.prototype 上的一个方法，类似于上个例子中 sayName() 与PersonType.prototype 之间的关系。这些相似处允许你把自定义类型与类混合使用，而不必被具体的用法困扰。

## 为何要使用类的语法

尽管类与自定义类型之间有相似性，但仍然要记住一些重要的区别：
1. 类声明不会被提升，这与函数定义不同。类声明的行为与 let 相似，因此在程序执行到声明处之前，类都会位于暂时性死区内。
2. 类声明中的所有代码会自动运行并锁定在严格模式下
3. 类的所有方法都是不可枚举的，这是对于自定义类型的显著变化，后者必须用 Object.defineProperty() 才能将方法改变为不可枚举。
4. 类的所有方法内部都没有 [[Construct]] ，因此使用 new 来调用它们会抛出错误。
5. 调用类构造器时不使用 new ，会抛出错误。
6. 试图在类的方法内部重写类名，会抛出错误。

这样看来，上例中的 PersonClass 声明实际上就直接等价于以下未使用类语法的代码：
```js
// 直接等价于 PersonClass
let PersonType2 = (function() {
  "use strict";
  const PersonType2 = function(name) {
    // 确认函数被调用时使用了 new
    if (typeof new.target === "undefined") {
      throw new Error("Constructor must be called with new.");
    }
    this.name = name;
  }
  Object.defineProperty(PersonType2.prototype, "sayName", {
      value: function() {
      // 确认函数被调用时没有使用 new
      if (typeof new.target !== "undefined") {
        throw new Error("Method cannot be called with new.");
      }
      console.log(this.name);
    },
    enumerable: false,
    writable: true,
    configurable: true
  });
  return PersonType2;
}());
```

首先要注意这里有两个 PersonType2 声明：一个在外部作用域的 let 声明，一个在 IIFE 内部的 const 声明。这说明了为何类名不能在类的方法内被重写，而允许在外部重写。构造器函数检查了 new.target ，以保证是使用 new 进行调用的，否则就抛出错误。接下来，sayName() 方法被定义为不可枚举，并且此方法也检查了 new.target ，它则要保证在被调用时没有使用 new 。最后一步是将构造器函数返回出去。

此例说明了不使用新语法也能实现类的任何特性，不过类语法显著简化了所有功能的代码。

只有在类的内部，类名才被视为是使用 const 声明的。这意味着你可以在外部重写类名，但不能在类的方法内部这么做。例如：
```js
class Foo {
  constructor() {
    Foo = "bar"; // 执行时抛出错误
  }
} // 但在类声明之后没问题
Foo = "baz";
```

在此代码中，类构造器内部的 Foo 与在类外部的 Foo 是不同的绑定。内部的 Foo 就像是用 const 定义的，不能被重写，当构造器尝试使用任何值重写 Foo 时，都会抛出错误。但由于外部的 Foo 就像是用 let 声明的，你可以随时重写类名。

## 类表达式

类与函数相似之处在于都有两种形式：声明与表达式。函数声明与类声明都以适当的关键词为起始（ 分别是 function 与 class ） ，随后是标识符（ 即函数名或类名） 。函数具有一种表达式形式，无须在 function 后面使用标识符；类似的，类也有不需要标识符的表达式形
式。类表达式被设计用于变量声明，或可作为参数传递给函数。

## 基本的类表达式

此处是与上例中的 PersonClass 等效的类表达式，随后的代码使用了它：
```js
let PersonClass = class {
  // 等价于 PersonType 构造器
  constructor(name) {
    this.name = name;
  }
  // 等价于 PersonType.prototype.sayName
  sayName() {
    console.log(this.name);
  }
};

let person = new PersonClass("Nicholas");
person.sayName(); // 输出 "Nicholas"
console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true
console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass.prototype.sayName); // "function"
```

正如此例所示，类表达式不需要在 class 关键字后使用标识符。除了语法差异，类表达式的功能等价于类声明。

使用类声明还是类表达式，主要是代码风格问题。相对于函数声明与函数表达式之间的区别，类声明与类表达式都不会被提升，因此对代码运行时的行为影响甚微

### 具名类表达式
上一节的示例使用了一个匿名的类表达式，不过就像函数表达式那样，你也可以为类表达式命名。为此需要在 class 关键字后添加标识符，就像这样：
```js
let PersonClass = class PersonClass2 {
  // 等价于 PersonType 构造器
  constructor(name) {
    this.name = name;
  }
  // 等价于 PersonType.prototype.sayName
  sayName() {
    console.log(this.name);
  }
};

console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass2); // "undefined"
```

此例中的类表达式被命名为 PersonClass2 。 PersonClass2 标识符只在类定义内部存在，因此只能用在类方法内部（ 例如本例的 sayName() 内） 。在类的外部， typeof PersonClass2
的结果为 "undefined" ，这是因为外部不存在 PersonClass2 绑定。要理解为何如此，请查看未使用类语法的等价声明：
```js
// 直接等价于 PersonClass 具名的类表达式
let PersonClass = (function() {
  "use strict";
  const PersonClass2 = function(name) {
    // 确认函数被调用时使用了 new
    if (typeof new.target === "undefined") {
      throw new Error("Constructor must be called with new.");
    }
    this.name = name;
  }
  Object.defineProperty(PersonClass2.prototype, "sayName", {
    value: function() {
      // 确认函数被调用时没有使用 new
      if (typeof new.target !== "undefined") {
        throw new Error("Method cannot be called with new.");
      }
      console.log(this.name);
    },
    enumerable: false,
    writable: true,
    configurable: true
  });
  return PersonClass2;
}());
```

创建具名的类表达式， JS 引擎的内部实现稍微有了变化。对于类声明来说，用 let 定义的外部绑定与用 const 定义的内部绑定有着相同的名称。而类表达式可在内部使用 const 来定义它的不同名称，于是此处的 PersonClass2 就只能在类的内部使用。

尽管具名类表达式的行为异于具名函数表达式，但它们之间仍然有许多相似点。二者都能被当作值来使用，存在多种利用可能，接下来我将会对此进行介绍。