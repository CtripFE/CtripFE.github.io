---
layout: post
title:  "编写现代 JavaScript 代码"
date:   2017-06-02 10:00:00
author: mirreal
categories: translator
---

> 原文作者：[Sébastien Castiel](https://twitter.com/scastiel)
> 
> 原文链接：[Writing modern JavaScript code](https://dev.to/scastiel/writing-modern-javascript-code)
>
> 说点什么：这是一篇很朴素的文章，讲的道理都懂，但实际上，在工作中遇到类似的情形却未必如此，编写可维护，可阅读，更安全的代码是我们应有的责任。

是不是还认为 JavaScript 是一门用于在光标悬浮时改变页面元素的语言？这些日子已经不复存在，每一种语言都在随着时间推移而发展，我们使用语言的方式同样也在发展。看一下你一两年前写的代码：会感到羞愧吗？如果是的话，这篇文章应该很适合你。

这里会列出一些所谓的最佳实践，目的是让你的 JavaScript 代码更容易编写，阅读和维护。

## 使用可格式化代码的 linter

第一个建议是使用 linter 工具，可以帮助你检查在不同文件是否遵守一致的规则，尤其是当不同开发人员在同一个项目上工作：缩进，括号中的空格，替换 `==` 为 `===` ...

但更重要的是，尽可能使用 linter 工具自动修复代码。[ESLint](http://eslint.org/) 就做得很好（带有 `--fix` 选项），而且与所有主流 IDE 完美集成，可以在保存时自动修复文件。

还可以使用 [Prettier](https://github.com/prettier/prettier)，不过这款工具更注重格式化而不是静态检查，但处理后的结果基本相同。

下一步将介绍与 linter 工具一起使用的规则：

## 为你的 linter 定制现代化的规则

如果不知道你的代码需要什么样的规则，可以参考：[StandardJS](https://standardjs.com/)。这是一个**非常**严格的 linter，无法修改配置，但里面的每一条规则已经越来越多地被社区接纳。比如：

* 使用 2 个空格缩进（我曾经使用 4 个空格，但实际使用起来 2 个空格很不错）
* 不使用分号（一开始可能会觉得奇怪，但几天后就再也回不去了）
* 在关键字（如 if）和花括号使用空格，在括号不使用空格
* [等等](https://standardjs.com/rules.html)。

StandardJS 是一个独立的 Node 模块，可以进行 lint 和修复代码，但如果要在现有的大型项目中使用，并且想要停用一些规则（因为有些地方可能需要作大量修改），还可以使用 [ESLint 预定配置](https://github.com/feross/eslint-config-standard)。比如，我就停用了规则 [no-mixed-operators](http://eslint.org/docs/rules/no-mixed-operators) 和 [import / no-webpack-loader-syntax](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/no-webpack-loader-syntax.md)。

## 使用 ES2015+ 的新特性

如果你在使用 JavaScript 开发，根本没办法不听说 ES2015 +（或 ES6，ES7 ...）的特性。有的已经是我离不开的：

* 箭头函数：对于函数式编程，比如写 `x => x * 2` 这样的函数非常有用（见下一点）
* 类：停止使用原型函数，使用类更酷炫（但不要滥用，JavaScript 比任何面向对象的语言好多了）
* 对数组和对象的操作：

```javascript
function doSomething() {
  const a = doSomethingElse()
  const b = doSomethingWithA(a)
  const otherResults = { c: '😺', d: '🐶' }
  return { a, b, ...otherResults } // equivalent to { a: a, b: b }
}
const { a, c, ...rest } = doSomething() // Also works with arrays!
// `rest` looks like { b: ..., d: '🐶' }
```

* 使用 `async/await` 编写更简单的异步处理：

```javascript
// Please try to write the same code with classic promises ;)
async function doSomething() {
  const a = await getValueForA()
  const b = await getValueForBFromA(a)
  const [c, d] = await Promise.all([
    // parallel execution
    getValueForC(), getValueForDFromB(b)
  ])
  const total = await calculateTotal(a, b, c, d)
  return total / 1000
}
```

想知道如何使用这些特性呢？[我的另一篇文章能给一些建议](https://blog.castiel.me/posts/002-use-the-coolest-es6-features-everywhere.html)。（顺便说一下，使用最新版本的 Node.js，可能不再需要 Babel 就能使用这些新特性）

## 使用函数式编程

函数式编程最近很热门，取得不少成就，而且不仅仅是在 JavaScript 中。为什么呢？函数式编程能使代码更具可预测性，确定性，更安全，一旦习惯这种方式，代码会更容易维护。这里有一些简单的建议：

首先，停止使用 for 循环，在大多数（可能是所有？）情况下根本不需要。例如：

```javascript
const arr = [{ name: 'first', value: 13 }, { name: 'second', value: 7 }]

// Instead of:
const res = {}
for (let i = 0; i < arr.length; i++) {
  const calculatedValue = arr[i].value * 10
  if (calculatedValue > 100) {
    res[arr[i].name] = calculatedValue
  }
}

// Prefer:
const res = arr
  .map(elem => ({ name: elem.name, calculatedValue: elem.value * 10 }))
  .filter(elem => elem.calculatedValue > 100)
  .reduce((acc, elem) => ({
    [elem.name]: elem.calculatedValue,
    ...acc
  }), {})
```

好吧，这实际上是一个非常极端的例子，对于不习惯函数式编程的人而言，可能看起来更加复杂。但我们可以稍微简化一下：

```javascript
const enrichElementWithCalculatedValue =
  elem => ({ name: elem.name, calculatedValue: elem.value * 10 })
const filterElementsByValue = value =>
  elem => elem.calculatedValue > value
const aggregateElementInObject = (acc, elem) => ({
  [elem.name]: elem.calculatedValue,
  ...acc
})
const res = arr
  .map(enrichElementWithCalculatedValue)
  .filter(filterElementsByValue(100))
  .reduce(aggregateElementInObject, {})
```

在这里，我们定义了三个函数，其功能基本上与其名字一致。第二个建议：创建局部函数（即使是在已经存在的函数中）来说明代码的功能，不需要使用注释。

注意，三个局部函数不修改它们的执行上下文。没有外部变量被修改，没有其他服务被调用...在函数式编程中，它们被称为*纯函数*。纯函数具有很大的优势：

* 很容易测试，因为从给定参数只有一个可能的结果，不管被调用了多少次;
* 无论应用状态如何，都能保证相同的结果;
* 应用状态在函数调用之前和之后保持不变。

所以我的第三个建议是：尽可能地使用纯函数。

## 其他的一些建议

* 习惯于使用异步代码，并多使用 promise，看看 [RxJS](http://reactivex.io/rxjs/) 的 observales（有[一个很棒的教程关于从函数式编程到响应式编程](http://reactivex.io/learnrx/)）
* 写测试！这应该是很明显的，但是据我所知很多项目都有未经测试的代码，尽管测试 JavaScript（前端或后端）并不困难。
* 使用最新的语言特性：比如不要再写 `arr.indexOf(elem) !== -1`，而应该写成 `arr.includes(elem)`。
* 大量阅读技术文章：[JavaScript subreddit](https://www.reddit.com/r/javascript/) 是了解目前社区最酷做法的一个很好的来源。

总而言之，最好的建议就是：**总是重构你的代码**。比如改进你一年前写过的模块？借此机会，用 `const` 取代 `var`，使用箭头函数或 `async/await` 简化代码......和你喜欢的代码工作一件很愉悦的事。
