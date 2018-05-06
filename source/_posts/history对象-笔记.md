---
title: history 对象 -- 笔记
date: 2018-04-12 10:31:39
tags:
---

## history 对象
### 1. 概述
  * 浏览器窗口有一个 history 对象，用来保存浏览历史，比如当前窗口先后访问了三个网址，那么 history.length 属性等于 3
  * history 对象提供了一系列方法，允许在浏览历史之间移动。
    - back()：移动到上一个访问页面，等同于浏览器的后退键。
    - forward()：移动到下一个访问页面，等同于浏览器的前进键。
    - go()：接受一个整数作为参数，移动到该整数指定的页面，比如 go(1) 相当于 forward()，go(-1) 相当于 back(), go(0) 相当于刷新当前页面。
    - 如果移动的位置超出了访问历史的边界，以上三个方法并不报错，而是默默的失败。
### 2. history.pushState()
  * HTML5 为 history 对象添加了两个新方法，history.pushState() 和 history.replaceState()，用来在浏览历史中添加和修改记录。
  * history.pushState 方法接受三个参数，依次为：
    - state：一个与指定网址相关的状态对象，popstate 事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填 null。
    - title：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填 null。
    - url：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。
  * 添加新记录后，浏览器地址栏立刻显示 example.com/2.html，但并不会跳转到 2.html，甚至也不会检查 2.html 是否存在，它只是成为浏览历史中的最新记录。这时，你在地址栏输入一个新的地址（比如访问 google.com)，然后点击了倒退按钮，页面的 URL 将显示 2.html；你再点击一次倒退按钮，URL 将显示 1.html。
  * 总之，pushState 方法不会触发页面刷新，只是导致 history 对象发生变化，地址栏会有反应。
  * 如果 pushState 的 url 参数，设置了一个新的锚点值（即 hash），并不会触发 hashchange 事件。如果设置了一个跨域网址，则会报错。

### 3. history.replaceState()
  * history.replaceState 方法的参数与 pushState 方法一模一样，区别是它修改浏览历史中当前纪录
### 4. history.state 属性
  * history.state 属性返回当前页面的 state 对象
### 5. popstate 事件
  * 调用 pushState 方法或 replaceState 方法 ，并不会触发该事件，只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用 back、forward、go 方法时才会触发。另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。
  * 使用的时候，可以为 popstate 事件指定回调函数。
  * 回调函数的参数是一个event事件对象，它的state属性指向pushState和replaceState方法为当前 URL 所提供的状态对象（即这两个方法的第一个参数）。
  * 注意，页面第一次加载的时候，浏览器不会触发 popstate 事件。