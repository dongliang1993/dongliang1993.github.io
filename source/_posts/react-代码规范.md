---
title: react 代码规范
date: 2017-12-04 14:45:50
tags:
---
1. 布尔变量或返回布尔值的函数应该以“is”，“has”或“should”开头。
```js
// Dirty
const done = current >= goal
// Clean
const isComplete = current >= goal
```
2. 函数命名应该体现做了什么，而不是是怎样做的。换言之，不要在命名中体现出实现细节。假如有天出现变化，就不需要因此而重构引用该函数的代码。比如，今天可能会从 REST API 加载配置，但是可能明天就会将其直接写入到 JavaScript 中。
```js
// Dirty
const loadConfigFromServer = () => {
  ...
}
// Clean
const loadConfig = () => {
  ...
}
```
3. 构建 React 应用程序时，应该遵循以下最佳实践：

  1. 使用小函数，每个函数具备单一功能，即所谓的单一职责原则（Single responsibility principle）。确保每个函数都能完成一项工作，并做得很好。这样就能将复杂的组件分解成许多较小的组件。同时，将具备更好的可测试性。
  2. 小心抽象泄露（leaky abstractions）。换言之，不要强迫消费方去了解内部代码实现细节。
4. 使用 defaultProps
5. 使用无状态组件
