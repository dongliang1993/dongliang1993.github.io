---
title: babel从入门到入门的知识归纳
date: 2017-12-04 15:18:12
tags:
---
1、babel是什么

2、javascript制作规范

3、babel转译器

4、babel的使用

5、常见的几种babel转译器和插件

6、babel最常见配置选项

7、babel的其他

8、在webpack中使用babel

9、总结

# 1、babel是什么

babel官网正中间一行黄色大字写着 “babel is a javascript compiler”，翻译一下就是babel是一个 javascript 编译器。

为什么会有babel存在呢？原因 javascript 在不断的发展，但是浏览器的发展速度跟不上。

以 es6 为例，es6 中为 javascript 增加了箭头函数、块级作用域等新的语法和 Symbol、Promise 等新的数据类型，但是这些语法和数据类型并不能够马上被现在的浏览器全部支持，为了能在现有的浏览器上使用 js 新的语法和新的数据类型，就需要使用一个转译器，将 javascript 中新增的特性转为现代浏览器能理解的形式。babel 就是做这个方面的转化工作。

# 2、javascript制作规范

在这里有必要简单讲一下 javascript 版本，我只是大体讲下。

javascript 是网景公司开发的一种脚本语言，1996 年的时候以 ECMAScript 的名字正式成为一种标准。2007 年的时候发布了版本 es5 ，然后在随后近 10 年里 js 并没有大的变化。所以现在的浏览器都可以很好的支持 es5。这一局面直到 2015 年被打破。2015年6月，TC39（javascript标准的制定组织）公布了新版本的js语言——ES6。而且从ES6开始，TC39规定每年都要发布一个js的新版本，新版本将包含年号，都是以ESxxxx的方式进行命名。所以2015年发布的ES6又叫ES2015，2016年发布的新的js版本就叫ES2016，2017年发布的新的js版本就叫ES2017……。

因为版本都是向前兼容的，就是老版本js版本中规定的语法和api在新版本的js中同样也会合理的。所以我们可以想到后面的规范肯定是包含前面的规范的，也就是ES2016版本的js规范是包含ES2015(ES6)规范的，ES2017是包含ES2016的也包含ES2015的。针对不同的规范，Babel也提供了对应的转换器。

babel-preset-es2015 将es2015版本的js转译为es5。
babel-preset-es2016 将es2016版本的js转译为es5。
babel-preset-es2017 将es2017版本的js转译为es5。
在转译过程中遇到更高版本的js语法，babel是会直接忽略的。
