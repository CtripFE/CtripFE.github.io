---
layout: post
title:  "喜欢用 Git 做的一些小事"
date:   2017-06-12 10:00:00
author: mirreal
categories: translator
---

> 作者：[@csswizardry](https://twitter.com/csswizardry)
>
> 原文：[Little Things I Like to Do with Git](https://csswizardry.com/2017/05/little-things-i-like-to-do-with-git/)
> 
> 随便说点：这篇文章主要从管理者的角度谈论了使用 git 的心得，使用大量篇幅介绍 `git log` 的一些方法和技巧。
> 
> 同样地，发现很多人其实并没有深入全面地去了解过 git 的用法，作为一名开发人员，大多数时候只要会使用 `git pull`，`git add`，`git commit`，`git push` 似乎就足够，还有很大一部分人只使用特定的图形化工具。但事实上真的是这样吗，可能在遇到某个稍微高级一点的问题或者需求就手足无措。对于某个特定个体而言，很多场景我们未必会遇到，即使碰到也可以现场寻求搜索工具的帮助，这也是一种学习方式，无意否定这种方式，但最大的问题就是只见一叶而难以窥见森林。在这里推荐一个小工具 [githug](https://github.com/Gazler/githug)，通过一种比较轻松的游戏的方式来一探全貌。

在跟我的朋友 Tim 聊天的时候，谈到我有多喜欢 Git。作为经常使用的一个工具，它强大而优雅。在这里，介绍一下我个人使用得最多，同时也是最有用的一些小技巧。

## 管理者面板

无论你认为在工作中的游戏化（gamification）和竞争是好是坏，对于这个话题在不同的时间可能是完全不同的结论。但如果你对团队成员在项目中的提交数量感兴趣，使用 `shortlog` 就可以找到答案：

（请忽略我）

（我只是占位的）


```sh
$ git shortlog -sn
    80  Harry Roberts
    34  Samantha Peters
     3  Tom Smith
```

`shortlog` 可以视作对 `git log` 的概要。

* `-s`选项将隐藏提交描述，仅提供提交计数摘要
* `-n` 选项将根据每个作者的提交数对输出进行排序，而不是默认的按作者字母顺序。

上面显示的是项目生命周期的所有提交，但是如果想查看在特定时间内的情况，可以使用 `--since` 和 `--until` 选项：

```sh
$ git shortlog -sn --since='10 weeks' --until='2 weeks'
    59  Harry Roberts
    24  Samantha Peters
```
我为此配置了别名 `$ git stats`

## 责任人

Git 有一个非常有用的 `blame` 功能，允许我们查看特定代码段的负责开发人员：

```sh
# See who last changed lines 5 through 10 of the buttons’ CSS:
$ git blame -L5,10 _components.buttons.scss
```

这一条放在这里讲好像有点过头，像是我们在找开发人员哪些地方做错了。但也不完全是这样，另一方面，他们可能已经做了一些我们想要了解的特别厉害或是印象深刻的事情。我们原本会问，哇！我之前还没有看到这个功能，还想知道是谁做的。

由于是从 SVN 转到 Git，我使用 `praise` 作为 `blame` 的别名，这样二者都可以使用：

```sh
$ git config --global alias.praise blame
```

即，我也可以这样做：

```sh
# Find out who implemented Resource Hints and buy them a coffee:
$ git praise -L18,23 _includes/head.html
```

只是一点小变化，但效果不错。

## 隐藏空白提示

当使用 `diff` 或 `show` 查看具有大量空白变化的版本对比时，会有很多视觉噪音干扰我们，使得很难看到更重要的变化内容。

幸运的是，去除这种空白提示非常容易，在 `git diff` 和 `git show` 使用 `-w` 选项就可以轻松搞定。比如，之前：

```
 a {
   color: $color-links;

-&:hover {
-  color: $color-links-hover;
-}
+  &:hover {
+    color: $color-links-hover;
+    text-decoration: underline;
+  }

}
```

使用 `-w` 之后：

```
 a {
   color: $color-links;

   &:hover {
     color: $color-links-hover;
+    text-decoration: underline;
   }

}
```

现在可以很容易看出，唯一有意义的变化是增加了 `text-decoration: underline;`，而其余​​的 `diff` 是有点误导性的。

## 仅显示单词的变化而不是整行

写代码跟写文章不同，查看变化的单词而不是整行通常会更有用; 这在编辑 markdown 文档时尤其有用，就像现在。

幸运的是，我们只要使用 `--word-diff` 选项就能显示单词的变化：

```sh
$ git diff --word-diff
```

跟不使用 `--word-diff` 选项的区别还是很大的：

```
-My friend Tom recently gave an excellent talk
+My good friend Tom gave an excellent talk
```

如果启用 `--word-diff`，我们能得到更便于理解和更有用的概览：

```
My {+good+} friend Tom [-recently-] gave an excellent talk
```

注意只有变化的文本被突出显示（通过 `{+ +}` 和 `[- -]`)

## 查看最近工作的分支

在任何给定的项目，在许多不同的分支之间切换是很常见的，并且跟踪它们可能相当棘手。我们可以让 Git 帮助我们解决这个问题：

```sh
$ git for-each-ref --count=10 --sort=-committerdate refs/heads/ --format="%(refname:short)"
```

通过这个命令可以知道最近在工作的 10（--count=10）个分支，按照上次工作的时间排序。只显示本地分支（`refs/heads/`），并通过 `--format` 选项获得更友好的呈现方式。

这是一个有点冗长的命令，所以我为此配置别名 `$ git recent`。

```sh
git config --global alias.recent "for-each-ref --count=10 --sort=-committerdate refs/heads/ --format=\"%(refname:short)\""
```

## 看到每个人都在做什么

有时候，特别是对于团队领导，了解团队成员在所有分支的行为概览是很有用的。再一次地，Git 可以让这一切变得很容易：

```sh
$ git log --all --oneline --no-merges
```

这可以得到一份关于所有人的日志报告简化版（带有 `--no-merges` 选项）

我们也可以通过 `--since` 选项来限制返回的提交数量：

```sh
$ git log --all --since='2 weeks' --oneline --no-merges
```

这样我们可以看到，在过去的两个星期里，每个人都在做什么。

可以配置一个别名 `$ git overview`

```sh
git config --global alias.overview "log --all --since='2 weeks' --oneline --no-merges"
```

## 提醒你自己已经做了什么

当你回到一个比较旧的项目，或是在长时间休息之后回到办公室，可能不知道你最后在做什么工作，这种情况时常发生。我们可以通过 Git 轻松获得我们在项目中的工作情况：

```sh
$ git log --all --oneline --no-merges --author=<your email address>
```

和上一条很类似，只是我们将日志限制于我们自己的提交，也可以增加 `--since` 限制。

这也有一个别名 `$ git recap`。

```sh
git config --global alias.recap "log --all --oneline --no-merges --author=name@mail.com"
```

## 今天的工作

同样地，不在这里讨论如何衡量开发人员的生产力，但我觉得让客户知道我在任何一天的工作情况是很有用的。不是要你保留完成任务的详细列表，我们可以使用 Git 获取所有这些信息：

```sh
$ git log --since=00:00:00 --all --no-merges --oneline --author=<your email address>
```

这将记录（`log`） 你工作的所有（`--all`）分支，谁（`--author`）从（`--since`）午夜开始都做了什么，（不包括合并提交 `--no-merges`），并提供一个简单的一行 （`--oneline`） 概述。

我有这个别名 `$ git today`。

```sh
git config --global alias.today "log --since=00:00:00 --all --no-merges --oneline --author=name@mail.com"
```

## 生成更改日志

维护一份 CHANGELOG 可能有点乏味，我们必须查看自上次发布以来所做的所有工作，然后提取其中有用的部分。幸运的是，我们可以使用 Git 来给我们一个好的开头：

```sh
$ git log --oneline --no-merges <last tag>..HEAD
```

注意：`HEAD` 是可选的，如果你省略（即... --no-merges <last tag>..），`HEAD` 会是隐含的，当然这样可以节省几次敲击键盘的时间。

这将创建一个简化的日志，显示最后一个发布版本和 `HEAD` 之间的所有提交（不包括合并提交）。

例如：

```sh
$ git log --oneline --no-merges 1.0.0..
1257b95 [refs #00019] Bump version
2b9b28e [refs #00019] Add auto width class
17b8eb1 [refs #00015] Tidy up README.md
bbe7d05 [refs #00012] Rename Supercell main mixin
```

这告诉我，自从上次发布（1.0.0）到当前项目状态（`HEAD`），已经完成哪些工作。这对于 CHANGELOG 来说是一个很好的参考。

注意：不仅仅适用于 tag，还可以使用提交哈希。

## 检查需要拉取哪些变化

如果你在一段时间内不在项目，可能需要先检查上游的变更，然后再将这些更新下载到本地分支。

```sh
$ git log --oneline --no-merges HEAD..<remote>/<branch>
```

注意：同样地，`HEAD` 在这里是可选的，省略将使其隐含。

例如，让我们来看看你在度假时在特性分支做了什么：

```sh
$ git checkout feature/fonts
$ git fetch
$ git log --oneline --no-merges ..origin/feature/fonts
```

我使用这个别名 `$ git upstream`。

## 检查即将上传的内容

最好的情况是可以经常提交和上传，但如果某种原因导致有大量的本地提交尚未上传，可以快速回顾一下都是什么。

为了做到这一点，我们反转之前的命令就能轻松实现：

```sh
$ git log --oneline --no-merges <remote>/<branch>..HEAD
```

例如：

```sh
$ git fetch
$ git log --oneline --no-merges origin/feature/fonts..HEAD
```

注意：同样地，`HEAD` 在这里是可选的，省略将使其隐含。

这将记录 `HEAD` 需要上传到 `<remote>/<branch>` 的提交。

我使用这个别名 `$ git local`。

## 查看复杂日志

上面的每一个例子都使用简化的日志，因为只想快速了解发生了什么。对于更多细节，我使用带有 `--graph` 选项的日志和一些额外的选项：

```sh
$ git log --graph --all --decorate --stat --date=iso
```

这将给出所有（`--all`）分支基于 `--graph` 的提交记录 ` --stat`（添加，删除）日志。`--decorate` 选项会告诉我们提交信息适用于那些分支，还包含一个更加严格的日期格式。

我使用这个别名 `$ git graph`。

```sh
git config --global alias.graph "log --graph --all --decorate --stat --date=iso"
```
