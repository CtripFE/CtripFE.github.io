---
layout: post
title:  "JS/React 开发者的 Atom 终极配置"
date:   2017-05-28 10:00:00
author: mirreal
categories: translator
---

> 原文作者：Elad Ossadon
>
> 原文链接：[The Ultimate Atom Editor Setup (+for JS/React)](https://medium.com/productivity-freak/my-atom-editor-setup-for-js-react-9726cd69ad20)

根据多年以来不断完善 Sublime Text 配置的经验，决定这次给 Atom 也来一个大改造。这个过程花费了好几个月的时间，但成果还是很卓有成效的，我现在非常满意这份配置。

这份清单将分为实用工具和 React 专用，并涉及到快捷键绑定。

## 实用工具

###  [atom-beautify](https://atom.io/packages/atom-beautify)

可以在 Atom 中 美化 HTML，CSS，JavaScript，PHP，Python，Ruby，Java，C，C ++，C＃，Objective-C，CoffeeScript，TypeScript，Coldfusion，SQL等。

快捷键：`⌃+⌥+B`

### [prettier-atom](https://atom.io/packages/prettier-atom)

使用 Prettier 来格式化 JavaScript 代码，配有强大的 ESlint 集成。

快捷键：`⌃+⌥+F`

### [atom-transpose](https://atom.io/packages/atom-transpose)

Atom 的转置更像是字符串翻转。在 Sublime 可以将选中的两个字符串进行交换，看起来更有用。

快捷键：`⌥+T`

### [case-keep-replace](https://atom.io/packages/case-keep-replace)

使用这个插件可以在替换文本的时候可以保留原来的命名风格。

快捷键：`⌘+⌃+R`

### [change-case](https://atom.io/packages/change-case)

一个可以快速改变当前选择文本命名方式的工具。比如可以从 `camelCase` 转换到 `snake_case` 等。

快捷键：`⌘+K ⌘+C/S`

### [copy-path](https://atom.io/packages/copy-path)

可以灵活地复制文件路径。

### [duplicate-line-or-selection](https://atom.io/packages/duplicate-line-or-selection)

重复选择文本或者重复一行，跟 Sublime Text 的行为一致，Atom 可以重复一整行。

快捷键：`⌘+⇧+D`

### [editorconfig](https://atom.io/packages/editorconfig)

帮助开发人员在不同的编辑器之间保持一致的编码风格。

### [file-icons](https://atom.io/packages/file-icons)

Atom 的文件特定图标插件，便于区分不同文件类型。

### [git-plus](https://atom.io/packages/git-plus)

VIM 风格的 git 插件，在没有终端命令行的时候进行提交等其他 git 操作。

### [highlight-selected](https://atom.io/packages/highlight-selected)

通过双击一个词来高亮整个文件相同的词。

### [local-history](https://atom.io/packages/local-history)

用于维护本地文件历史的插件（对代码文件进行更改的历史记录）。

### [project-manager](https://atom.io/packages/project-manager)

轻松访问所有项目，还能对项目特定设置和选项进行管理。

快捷键：`⌘+⌃+P`

### [atom-reveal-file-in-finder](https://atom.io/packages/atom-reveal-file-in-finder)

可以在工作区或者文件选项卡上打开文件到 Finder 上，快捷命令已经添加到 `⌘+⇧*+P` 。

快捷键：`⌘+⌃+P`

### [related](https://atom.io/packages/related)

related 提供了访问与当前打开的文件相关的文件的快速方式。 例如，在 `.js` 和 `.spec.js` 文件之间切换。

快捷键：`⌘+⇧+ R`

我的 JS 关联配置 (Menu > Packages > Related > Edit related patterns):

```
'([^\\/]+)(?!\\.spec).js(x?)$': [
  'tests/$1.spec.js$2#create',
]
'tests/(.+).spec.js(x?)$': [
  '$1.js$2',
]
```

### [set-syntax](https://atom.io/packages/set-syntax)

一种简单的命令方式来设置当前文件语法，与 Sublime 类似。

快捷键：`⌘+⇧+P`

### [sort-lines](https://atom.io/packages/sort-lines)

排序/删除重复行。

### [sublime-style-column-selection](https://atom.io/packages/Sublime-Style-Column-Selection)

alt +单击跨行选择文本块，每行都有插入符号。

快捷键：`⌥+Drag`

### [tab-foldername-index](https://atom.io/packages/tab-foldername-index)

可以替换 TAB 标签内容的插件，在打开相同文件名的文件时保证更高的可读性。

### [sync-settings](https://atom.io/packages/sync-settings)

跨 Atom 实例同步设置，键盘映射，用户样式，初始化脚本，代码段和已安装的软件包。 我将所有设置备份到 Gist 并在工作/个人计算机之间进行同步。

### [toggle-quotes](https://atom.io/packages/toggle-quotes)

快速切换字符串的单引号和双引号。

快捷键：`⌘+⇧+’`

### [atom-spotify2](https://atom.io/packages/atom-spotify2)

在 Atom 状态栏中显示在 Spotify 中当前播放歌曲。 不是必要的，但很有趣。

## HTML/CSS/JS/React Specific Packages

### [atom-ternjs](https://atom.io/packages/atom-ternjs)

使用 Tern 为 Atom 提供 JavaScript 代码智能提示，支持 ES5，ES6，ES7，Node.js，jQuery，Angular等。 

### [atom-wrap-in-tag](https://atom.io/packages/atom-wrap-in-tag)

为选择的文本增加标签。

快捷键：`⌥+⇧+W`

### [autoclose-html](https://atom.io/packages/autoclose-html)

自动添加关闭标签。

### [autocomplete-modules](https://atom.io/packages/autocomplete-modules)

自动补全 `require/import` 声明。

### [color-picker](https://atom.io/packages/color-picker)

很厉害的颜色选择器。

快捷键：`⌘+⇧+D`

### [docblockr](https://atom.io/packages/docblockr)

更容易的方式写文档注释。

使用方式：` /** <tab>`

### [emmet](https://atom.io/packages/emmet)

一个大大提高 HTML 和 CSS 工作流程的插件。 [关于 Emmet](http://emmet.io/)

### [emmet-jsx-css-modules](https://atom.io/packages/emmet-jsx-css-modules)

适用于 css 模块的 emmet 工具。 `.foo` 现在将扩展为 `<div className = {style.foo}> </ div>`，而不是 `<div className =“foo”> </ div>`。

### [es6-javascript](https://atom.io/packages/es6-javascript)

一组专注 ES6，用于优化现代 JavaScript 开发生产力的命令集， 目标是符合 [Airbnb 推荐的代码规范](https://github.com/airbnb/javascript)。

### [js-hyperclick](https://atom.io/packages/js-hyperclick) & [hyperclick](https://atom.io/packages/hyperclick)

点击跳到变量或者 import 定义，js-hyperclick 依赖于 hyperclick。

### [pigments](https://atom.io/packages/pigments)

在项目文件中显示颜色。

### [linter-eslint](https://atom.io/packages/linter-eslint)

插件 [Linter](https://github.com/AtomLinter/Linter) 为 [eslint](http://eslint.org/) 提供 UI 接口，用于对 JavaScript 文件进行静态检查。

### [tree-view-copy-relative-path](https://atom.io/packages/tree-view-copy-relative-path)

允许从 tree view 复制文件的相对路径。

### [lodash-snippets](https://atom.io/packages/lodash-snippets)

在 Atom 中快速使用 lodash 的代码提示。

### [language-babel](https://atom.io/packages/language-babel)

支持 JavaScript ES201x，React JSX，Flow和GraphQL语法。

### [react-es7-snippets](https://atom.io/packages/react-es7-snippets)

React ES7 snippets for atom

### [atom-jest-snippets](https://atom.io/packages/atom-jest-snippets)

Jest 测试提示

## 我的主题

### UI Theme: [one-dark-ui](https://atom.io/themes/one-dark-ui)

### Syntax Theme: [dracula-theme](https://atom.io/themes/dracula-theme)

## Install EVERYTHING!

```
apm install atom-beautify prettier-atom atom-spotify2 atom-transpose case-keep-replace change-case copy-path duplicate-line-or-selection editorconfig file-icons git-plus highlight-selected local-history project-manager related set-syntax atom-reveal-file-in-finder sort-lines sublime-style-column-selection tab-foldername-index sync-settings toggle-quotes atom-wrap-in-tag atom-ternjs autoclose-html autocomplete-modules color-picker docblockr emmet emmet-jsx-css-modules es6-javascript js-hyperclick hyperclick pigments linter-eslint tree-view-copy-relative-path lodash-snippets language-babel react-es7-snippets atom-jest-snippets one-dark-ui dracula-theme
```