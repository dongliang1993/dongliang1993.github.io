---
title: for in 循环顺序的问题
date: 2018-01-25 15:52:19
tags:
---

for...in 循环出来的对象属性顺序归结成一句话就是：先遍历出整数属性（integer properties，按照升序），然后其他属性按照创建时候的顺序遍历出来。

我们来看一个例子：
```js
let codes = {
  "49": "Germany",
  "41": "Switzerland",
  "44": "Great Britain",
  "1": "USA"
};

for (let code in codes) {
  alert(code); // 1, 41, 44, 49
}
```

最终遍历出来的结果是：属性 1 先遍历出来， 49 最后遍历出来。

这里的 1、41、44 和 49 就是整数属性。
那什么是整数属性呢？我们可以用下面的比较结果说明：
```js
String(Math.trunc(Number(prop)) === prop
```
当上面的判断结果为 true，prop 就是整数属性，否则不是。

所以

* "49" 是整数属性，因为 String(Math.trunc(Number('49')) 的结果还是 "49"。
* "+49" 不是整数属性，因为 String(Math.trunc(Number('+49')) 的结果是 "49"，不是 "+49"。
* "1.2" 不是整数属性，因为 String(Math.trunc(Number('1.2')) 的结果是 "1"，不是 "1.2"。

上面的例子中，如果想按照创建顺序循环出来，可以用一个 讨巧 的方法：
```js
let codes = {
  "+49": "Germany",
  "+41": "Switzerland",
  "+44": "Great Britain",
  // ..,
  "+1": "USA"
};

for(let code in codes) {
  console.log( +code ); // 49, 41, 44, 1
}
```