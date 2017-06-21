---
layout: post
title:  "JavaScript 模块化现状"
date:   2017-05-29 10:00:00
author: mirreal
categories: translator
---

> 原文作者：Johannes Ewald [@Jhnnns](https://twitter.com/Jhnnns)
> 
> 原文链接：[The state of JavaScript modules](https://medium.com/webpack/the-state-of-javascript-modules-4636d1774358)
> 
> 已获原作者授权翻译及发布

ESM, CJS, UMD, AMD — 到底应该选择哪一个？

最近 [在 twitter](https://twitter.com/addyosmani/status/859296190323597313) 上有很多关于 [ES Module](http://2ality.com/2014/09/es6-modules-final.html) 现状的讨论，[尤其是在 Node.js 上](https://twitter.com/bradleymeck/status/863061949021650944)，他们计划引入新的文件扩展名 `*.mjs`。人们有足够理由对此感到 [担忧和不确定](https://twitter.com/tankredhase/status/861864123922907136)，因为这个话题异常复杂，接下来会尽力阐述清楚问题。

## 来自远古的恐惧

大多数前端开发者应该还记得 [Javascript 依赖管理的黑暗时期](https://medium.com/@trek/last-week-i-had-a-small-meltdown-on-twitter-about-npms-future-plans-around-front-end-packaging-b424dd8d367a)。那个时候，你需要把一个库复制粘贴到 vendor 文件夹，然后作为一个全局变量引入，要自己去按次序组合所有东西，可能还要管理命名空间。

在过去的那些年，我们能深刻体会到公共模块格式化和中央模块管理的价值。

在今天，不管是发布还是使用一个库都要容易得多，只需要使用 `npm publish` 和 `npm install` 命令就行。这就是人们会那么紧张两种模块系统兼容性问题的原因：他们不想失去已有的舒适区。

接下来我会解释和总结现有实现的情况，以及为什么 Node 生态迁移到 ES Module（ESM）会那么难。在最后，总结这些变化对 webpack 使用者和模块作者有什么影响。

## 现有实现

目前，ESM 有三种方式的实现：

* 浏览器
* webpack 以及类似的构建工具
* Node（未完成，[但可能在年底作为一个实验功能](https://twitter.com/rauschma/status/866334160218095617)）

为了更好地理解现在的讨论，首先要知道 ES2015 包含两种模式：

* `script` 用于具有全局命名空间的常规脚本
* `module` 用于具有明确导入和导出的模块化代码

如果你试图在 `script` 标签使用 `import` 或者 `export` 语句，会抛出一个 SyntaxError。这种语句在全局环境下没有任何意义。另一方面，`module` 模式即意味着[严格模式](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Strict_mode)，禁止使用某些语言特性，比如 `with` 语句。因此，需要在脚本被解析和执行之前定义模式。

## 浏览器中的 ESM

截至到 2017 年 5 月，所有主流浏览器都开始做了 ESM 的实现工作。不过，大部分仍处于在实验性质。这里不会做详细介绍，因为 [Jake Archibald 已经写了一篇很厉害的文章](https://jakearchibald.com/2017/es-modules-in-browsers/)。

除了一些小的困难，在浏览器中实现起来非常容易，因为以前并没有模块系统。想要指定 `module` 模式，需要在 `script` 标签添加 `type="module"` 属性，如下所示：

```html
<script type="module" src="main.js"></script>
```

在一个模块中，现在只能使用有效的 `URL` 作为模块标识符。模块标识符是用于 require 或 import 其他模块的字符串。为了确保未来兼容 CJS 模块标识符，“纯” 导入标识符（如 `import "lodash"`）现在还不支持。模块标识符必须是绝对 `URL` 或者是以 `/`，  `./`,  `../` 开头：

```javascript
// Supported:
import {foo} from 'https://jakearchibald.com/utils/bar.js';
import {foo} from '/utils/bar.js';
import {foo} from './bar.js';
import {foo} from '../bar.js';

// Not supported:
import {foo} from 'bar.js';
import {foo} from 'utils/bar.js';
// Example from https://jakearchibald.com/2017/es-modules-in-browsers/
```

同样需要注意的是，一旦处在一个模块中，每个导入也将被解析为 `module`，而且没有办法 `import` 一个 `script`。

## ESM 与 webpack

类似 webpack 这样的构建工具通常会尝试用 `module` 模式解析代码，有问题再切回到 `script` 模式。这些工具最终会生成一段 `script`，通常是在一定程度上模拟 CJS 和 ESM 行为的模块运行时。

我们以这两个简单的	ESM	为例：

```javascript
// a.js
export let number = 42;
export function incr() {
    number++;
}
```

```javascript
// test.js
import { number } from "./a";

console.log(number);
```

webpack 使用函数包装器封装模块范围和对象引用来模拟 [ESM 实时绑定](http://2ality.com/2015/07/es6-module-exports.html)。每次编译，还包括一个模块运行时，负责引导和缓存模块。此外，将模块标识转换为数字模块 ID。这样可以减少打包的大小和引导时间。

这是什么意思呢？我们来看看编译输出：

```javascript
(function(modules) {
    // This is the module runtime.
    // It's only included once per compilation.
    // Other chunks share the same runtime.
    var installedModules = {};
    // The require function
    function __webpack_require__(moduleId) {
        ...
    }
    ...
    // Load entry module and return exports
    return __webpack_require__(__webpack_require__.s = 1);
})
([ // An array that maps module ids to functions
    // a.js as module id 0
    function (module, __webpack_exports__, __webpack_require__) {
        "use strict";
        Object.defineProperty(__webpack_exports__, "a", {
            configurable: false,
            enumerable: true,
            get: () => number
        });

        let number = 42;

        function incr() {
            number++;
        }
    },
    // test.js as module id 1
    function (module, __webpack_exports__, __webpack_require__) {
        "use strict";
        var __WEBPACK_IMPORTED_MODULE_0__a__ = __webpack_require__(0);

        // Object reference as "live binding"
        console.log(__WEBPACK_IMPORTED_MODULE_0__a__["a" /* number */]);
    }
]);
```

简化的 webpack 输出，模拟 ES Modules 行为

结果已经简化并删除了一些与此示例无关的代码。你会发现，webpack 在 `exports` 对象上将所有 `export` 语句替换成 `Object.defineProperty`，并使用属性访问器替换对引入值的所有引用。还要注意每个 ESM 开始时的 `"use strict"` 指令，这是由 webpack 自动添加，在 ESM 中必须是严格模式。

这种实现只是模拟，因为它试图模仿 ESM 和 CJS 的行为 -- 但不是与其完全保持一致。比如，这种模拟并不符合某些边缘情况。看下面这个模块：

```javascript
console.log(this);
```

如果你通过加上 `babel-preset-es2015` 的	Babel 来运行，结果是：

```javascript
"use strict";
console.log(undefined);
```

从输出结果可以看出，Babel 假设默认是 ESM，因为 `module` 模式即代表严格模式，在严格模式下会将 `this` 初始化为 `undefined`。

然而，使用 webpack，结果是：

```javascript
(function(module, exports) {

console.log(this);

})
```

在引导模块时，`this` 将指向 `exports` ，与 Node.js 使用的 CJS 行为一致。这是因为语法上不确定是 `script` 还是 `module`，解析器无法判断该模块是 ESM 还是 CJS。在不明确的时候，webpack 会模拟 CJS，因为它仍然是最受欢迎的模块风格。

这种模拟其实已经包含了很多情况，因为模块作者通常会避免这种代码。然而，“很多情况”对于像 Node.js 这样的平台是不够的，因为它需要保证所有有效的 JavaScript 代码都能正常运行。

## Node.js 中的 ESM

Node.js 在执行 ESM 时遇到了麻烦，因为仍然需要支持 CJS，语法看起来相似，但运行时行为完全不同。[Node.js 核心技术委员会](https://github.com/nodejs/CTC)（CTC）成员 James M Snell 撰写了[一篇很好的文章来解释 CJS 与 ESM 之间的差异](https://hackernoon.com/node-js-tc-39-and-modules-a1118aecf95e)。

归结起来，CJS 是一个动态模块系统，ESM 是静态模块系统。

### CJS

* 允许动态同步 `require()`
* 导出仅在模块执行后才知道
* 导出可以在模块初始化后添加，替换和删除

### ESM

* 只允许静态同步 `import`
* 在模块执行之前，导入和导出已经关联
* 导入和导出是不可变的

由于	CJS 早于 ES2015，所以一直在 `script` 模式下解析，封装通过使用函数包装器实现。在 Node.js 中加载 CJS，实际上会执行与此类似的代码：

```javascript
const module = {
    exports: {}
};
const require = makeRequireFunction();
const filename = "...";
const dirname = "...";
(function (exports, require, module, __filename, __dirname) {
/* YOUR CODE */
})(module.exports, require, module, filename, dirname);
```

对 Node.js 的 CommonJS 模块的简单函数包装

问题出现了，将两个模块系统集成到同一个运行时时，ESM 和 CJS 之间的循环依赖可能会迅速导致类似死锁的情况。

而且，由于现有 CJS 模块数量庞大，也不能直接放弃对 CJS 的支持。为了避免 Node.js 生态的中断，有两点已经很明显：

* 现有的 CJS 代码必须以相同的方式继续工作
* 两个模块系统都必须同时且尽可能无缝地工作

### 目前的权衡

2017 年 3 月，经过几个月的讨论，CTC 终于找到了一种解决问题的办法。由于在 ES 规范和引擎不改变的情况下无法进行无缝集成，[CTC 决定开始一些权衡之后的实现工作](https://github.com/bmeck/node-eps/blob/a1eab9bf023bbe13a79ddb18f0622a5d57215f9b/002-es-modules.md)：

#### 1.ESM 必须是 `*.mjs` 文件扩展名

这是由于上面提及的模糊语法问题，无法通过解析来确切知晓 JavaScript 代码是什么类型。为了 Node.js 向后兼容的目标，作者需要加入一种新模式。[已经有关于各种替代品的讨论](https://github.com/nodejs/node/wiki/ES6-Module-Detection-in-Node#detection-problem)，但使用不同文件扩展名是解决目前问题的最佳权衡。

#### 2.CJS 只能异步导入 ESM import()

Node.js 将异步加载 ESM，以便尽可能接近浏览器的行为。因此，同步的 `require()` 在 ESM 是不可能的，并且依赖于 ESM 的每个功能都需要异步：

```javascript
const driverPromise = import("dbdriver");

exports.readFromDb = async (query) => {
   return (await driverPromise).read(query);
};
```

#### 3. CJS 向 ESM 暴露一个不可变的默认导出

使用 Babel 或 Webpack，我们通常将 CJS 重构为 ESM，如下所示：

```javascript
// CJS
const { a, b } = require("c");
// ESM
import { a, b } from "c";
```

再一次地，他们的语法看起来很相似，但忽略了 CJS 中没有命名导出的事实。只有一个叫做 `default` 的导出，等同于在 CJS 模块完成计算后一个不可变的 `module.exports` 。从技术上讲，有可能将 `module.exports` 解构成命名导入，但这需要对标准作更大的变更。[这就是现在 CTC 决定采取这种方式的原因](https://github.com/bmeck/node-eps/blob/a1eab9bf023bbe13a79ddb18f0622a5d57215f9b/002-es-modules.md#461-default-imports)。

#### 4.模块范围的变量类似 `module`，`require` 以及 `__filename` 在 ESM 不存在

Node.js 和浏览器会实现一些 ESM 的特性，[但标准化过程仍在进行中](https://github.com/whatwg/html/issues/1013)。

鉴于将 CJS 和 ESM 集成到一个运行时的工程挑战，CTC 在评估边缘情况和权衡方面做了非常好的工作。比如使用不同的文件扩展名是就是一个很简单的解决方案。

实际上，一个文件扩展名可以认为是一个二进制文件如何解释的提示。如果一个 `module` 不是 `script`，我们应该使用不同的文件扩展名。其他工具（如 linter 或 IDE ）也可以获取相同信息。

当然，引入新的文件扩展名有成本，但是一旦服务器和其他应用程序确认 `*.mjs` 为JavaScript，我们很快就会忘记这个争议。

## 将 * .mjs 作为 Node.js 的 Python 3？

考虑到所有这些限制，人们可能会问，这种过渡将对现在的生态造成什么样的损害。虽然 CTC 会努力解决问题，但社区如何采用这一点仍然存在很大不确定性。这种不确定性 [被众多知名的 NPM 模块作者](https://twitter.com/sindresorhus/status/861987349529452545) 再次强调，他们声称将不会在模块中使用 `*.mjs`。

[Python 3 is killing Python](http://blog.thezerobit.com/2014/05/25/python-3-is-killing-python.html)

很难预测社区如何反应，但是应该不会对现在的生态造成大破坏，甚至能看到从 CJS 平稳过渡到 ESM。主要有两个原因：

### 1.与 CJS 严格向后兼容

那些不喜欢 ESM 的模块作者可以继续使用 CJS，保证自己不被排挤出局。这样他们自己的代码不会受到采用 ESM 的影响，降低迁移到另一个运行时的可能性，让 NPM 迁移到新生态变得容易。从 CJS 到 ESM 的重构给包维护者带来额外工作，不能指望所有人都有时间。

### 2. CJS 在 ESM 中的无缝整合

从 ESM 导入 CJS 模块非常简单。需要注意的是，CJS 仅导出一个默认值。一旦处于 ESM，甚至可能根本不会注意到依赖关系使用的模块风格，尤其是与在 CJS 中使用 `await import()`相比。

由于 ESM 的这个优点以及其他有点，比如开箱即用的 [tree shaking](https://webpack.js.org/guides/tree-shaking/) 和浏览器兼容性，预计在未来几年内，我们可以看到向 ESM 的缓慢而稳定的过渡。CJS 的特性，如动态 `require()` 和猴子补丁导出，在 Node.js 社区一直是有争议的，不比 ESM 带来的好处。

## 这些对我来说意味着什么？

因为最近这些事情，很容易对目前存在的所有选择和限制感到困惑。在接下来，整理了开发人员面临的典型问题以及我们的回答：


### 现在需要重构现有的代码吗？

**不需要**。Node.js 才刚刚开始实现 ESM，仍然有大量的工作要做。[James M Snell 预计至少还需要一年时间](https://medium.com/the-node-js-collection/an-update-on-es6-modules-in-node-js-42c958b890c)，还有很多变化的余地，所以现在重构是不安全的。

### 应该在新代码中使用 ESM 吗？

* **如果你已经有或者打算使用像 webpack 这样的构建工具，答案是肯定的**。这将更容易完成代码库的过渡，并使 tree shaking 成为可能。但要小心：一旦 Node.js 支持原生 ESM，可能需要重构其中的一些部分。
* **如果你正在编写一个库，答案是也肯定的**，你的模块使用者将受益于 tree shaking。
* **如果你不想进行构建操作，或者正在编写一个 Node.js 应用程序，还是用 CJS 吧**。

### 现在应该使用 .mjs 吗？

**不要这样做**，目前没有什么好处，工具支持依然薄弱。建议一旦原生 ESM 支持登陆 Node.js，尽快开始迁移。记住，[浏览器只关心 MIME 类型，而不是文件扩展名](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)。

### 应该关心浏览器兼容性吗？

是的，需要在一定程度上关注这个问题。 不应该在导入语句中省略 `.js` 扩展名，因为浏览器需要完整的 URL，无法像 Node.js 这样执行路径查询。同样，应该避免 `index.js` 文件。不过，人们并不会很快在浏览器中使用 NPM 软件包，因为仍然不能 bare 导入。

### 作为库作者该怎么办？

用 ESM 编写代码，并使用 [Rollup](https://rollupjs.org/) 或 Webpack 转换成单个 CJS 模块，然后在 `package.json` 将 `main` 字段指向此 CJS 包，并将 [`module`](https://github.com/rollup/rollup/wiki/pkg.module) 字段指向原始 ESM。如果还使用 ESM 之外的其他新语言功能，则应编译成 ES5，并提供 CJS 和 ESM 的打包。这样，你的库用户仍然可以从 tree shaking 获利而无需对代码进行转换。

看一下这些完成 tree shaking 的模块

## 总结

关于 ES 模块有很多不确定性。由于目前 Node.js 在实现上的权衡，开发人员担心可能会破坏 Node.js 的生态。

这些还不会发生，有两个原因：**CJS 的严格的后向兼容和 CJS 在 ESM 中的无缝集成**。

在 Node.js 发布原生 ESM 支持之前，应该仍然使用 Rollup 和 Webpack 等工具。它们在一定程度上模拟了 ESM 环境，但要注意它们不完全符合规范。此外，使用打包仍然是个[很好的选择](https://peerigon.github.io/talks/2016-08-26-jsconf-is-future-frontend-tooling/#36)，一旦可以在浏览器中使用 NPM 软件包。

我们 webpack 团队正在努力做一些工作，帮助开发者平稳过渡。为了这个目标，我们计划在 Node.js 的 ESM 支持成熟后，模拟 Node.js 导入 CJS 的方式。
