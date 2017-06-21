---
layout: post
title:  "移除在 ESM 模式生成的严格模式"
date:   2017-06-05 10:00:00
author: mirreal
categories: team
---

## 起源

项目升级构建方式，以前的代码采用 AMD 风格组织代码，并使用 `r.js` 完成打包优化工作。后续成员选择采用 ES6 的风格编写代码，然后 webpack + babel 完成打包构建。迁移工作并不如想象中那么容易，终于完成各种配置，测试时候发现问题：抛出一个语法错误。错误很快被定位，属于历史代码的不规范写法，之所以现在暴露出来是因为新的构建方式会加入 `"use strict"`。

## 严格模式

在编写代码的过程中，我们使用 `strict mode` 是一种很好的方式，严格模式会将 JavaScript 陷阱直接变成明显的错误，比如未声明导致的全局变量，可以让我们开发过程中就发现错误。

众所周知，JavaScript 这门语言之前存在大量不好的设计，使用严格模式意味着在采用一种限制性更高的方式编写代码，同时更 “安全”。但这里所谓的安全在生产环境又可能是另外一回事，尤其是对于那些上了年纪的历史代码，我们更倾向于在生产环境去除严格模式。

## ES2015+

现在大部分人已经在使用更强大，更具有表现力的 `ES2015+` 编写代码。可能还会使用到 babel 来转化成用于生产环境运行的代码，在使用 `preset es2015` 时，会自动加入 `“use strict”`。

babel 的处理方式是将 ES2015 模块转换成 `CommonJS` 格式的，然后再统一处理，即 `transform-es2015-modules-commonjs`，这个插件位于 `preset es2015` 中，并且依赖于 `transform-strict-mode`

## 也许你会想去掉严格模式

也许你会想去掉严格模式，毕竟对于一些历史代码，很难预测加入严格模式会导致什么异常。

<!-- 可以从两个方面来考虑： -->

* 第一，在处理过程中去掉 `"use strict"`，这里不用我们自己去写了，借助这个插件 `transform-remove-strict-mode`
* 第二，不用 `transform-es2015-modules-commonjs`，可以使用 `webpack 2` 直接处理 ES6 模块

### 方案一

```sh
npm install babel-plugin-transform-remove-strict-mode --save-dev
```

修改 `.babelrc`

```json
{
    "presets": [
        ["es2015"],
        "stage-0",
        "react"
    ],
    "plugins": [
        "transform-remove-strict-mode"
    ]
}
```

### 方案二

> [modules](https://babeljs.io/docs/plugins/preset-es2015/#optionsmodules)
>
> `"amd" | "umd" | "systemjs" | "commonjs" | false`, defaults to `"commonjs"`.
>
> Enable transformation of ES6 module syntax to another module type.
>
> Setting this to `false` will not transform modules.

我们去掉 `preset es2015` 的模块处理，改由 `webpack` 来处理：

```json
{
    "presets": [
        ["es2015", { "modules": false }],
        "stage-0",
        "react"
    ]
}
```

这种方案也能解决之前问题，因为之前存在问题的模块使用 ES5 编写的，使用 AMD 风格。

但是，参考 [http://www.ecma-international.org/ecma-262/6.0/#sec-strict-mode-code](http://www.ecma-international.org/ecma-262/6.0/#sec-strict-mode-code) ，在 ES6 语法下，模块系统以及 class 等仍然都是工作在 `strict mode` 下的，而 webpack2 的也确实是这样处理的。

实现对语言本身是没问题的，使用新语法意味着我们要抛弃掉以前一些设计不好的地方，编写高质量的代码。

### 对比

相比而言，方案二更合适，因为方案一处理的过程实际是删除所有`"use strict"`，有时候未必符合你本来的意愿。

采用方案二还有一些好处，比如可以使用 `webpack2` 已经支持的 `tree shaking` 优化技术，因为这项技术基于 `ES6 Modules`，得让 `webpack` 直接处理才能使用。

## 后续

这种属于历史遗留问题，比如不小心引入的全局变量，类似的代码质量问题还不少。

所以，我们使用 lint 工具来帮助我们避免这种问题，对于代码质量的要求必须苛刻。