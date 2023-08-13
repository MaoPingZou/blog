---
layout: post
title: 【Vim】IDEA中Vim插件结合EasyMotion插件高级玩法
subtitle: 强强联合，变成无情跳转机器！
tags: [idea, vim, tool]
---

> 本文将介绍**IDEA**中的**IdeaVim**插件提供的**EasyMotion**拓展插件。

目录
- [什么是EasyMotion？](#什么是easymotion)
- [怎么玩EasyMotion？](#怎么玩easymotion)
  - [安装](#安装)
  - [了解下EasyMotion的原理（重要）](#了解下easymotion的原理重要)
  - [基本使用](#基本使用)
- [最后](#最后)

### 什么是EasyMotion？

**EasyMotion**起源是**Vim**的一个插件，正如它的名字所表明的一样，**EasyMotion**可以让你在**Vim**中以更简单的方式移动。

一旦熟练掌握**EasyMotion**的使用，你将会惊讶自己使用键盘移动光标的速度竟然可以得到如此大的提升！



### 怎么玩EasyMotion？

#### 安装

只需要执行下面两步：

- 到**IDEA**插件市场去安装两个插件： [IdeaVim-EasyMotion](https://plugins.jetbrains.com/plugin/13360-ideavim-easymotion/)、 [AceJump](https://plugins.jetbrains.com/plugin/7086-acejump/)

- 往`.ideavimrc`中加一行代码：``Plug 'easymotion/vim-easymotion'``

然后重启一下**IDEA**，就可以开始愉快的使用**EasyMotion**了。

#### 了解下EasyMotion的原理（重要）

在使用**EasyMotion**之前，了解一下它工作的大致原理，会让你对它建立一个初步的概念，这对于后续使用它是很有帮助的。

> 在使用**EasyMotion**之前，你需要通过执行一个命令来激活它。**EasyMotion**的作者们把这个命令称作`leader`。默认的`leader`是`\`，但是你可以自定义其他更适合自己习惯的键位作为`leader`。
>
> 在使用`leader`激活**EasyMotion**之后，你就需要告诉它你想要做什么动作。**EasyMotion**的作者们把这称作`motion`。比如`w`表示触发单词移动动作，`s`表示触发搜索移动动作等等。
>
> 在你告诉**EasyMotion**想要做什么动作之后，它会高亮所有可能的跳转目标出来，然后，你就可以一键直接跳转到目标位置。

放一张官网的图，可以更形象的了解上面的文字：

![easymotion-office-demo-gif](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/vim-easymotion.gif)

上图使用的命令是：`\ssfk`。这条命令所代表的含义是：`\`激活**EasyMotion**，紧跟着的`s`表示触发搜索移动动作，第二个`s`表示搜索 s 这个字符，最后的`fk`表示对应的跳转目标。

是不是很好理解？迫不及待的想要尝试下！

#### 基本使用

这里介绍一下**EasyMotion**基本特性的使用。从这个简单的例子中你就能体会到**EasyMotion**的强大。

(下面例子中`<crusor>`表示光标所在位置)

```
<cursor>Lorem ipsum dolor sit amet.
```

按下`<Leader><Leader>w`来触发单词移动动作 `w`，当这个移动动作被触发时，文本将会更新（正如上面动态图上所看到的）。

```
<cursor>Lorem {a}psum {b}olor {c}it {d}met.
```

此时，按下 `c `会跳到单词 “sit” 的开头位置，如下：

```
Lorem ipsum dolor <cursor>sit amet.
```

同样，如果你想要寻找 `o` ，你可以使用 `f` 移动动作。按下`<Leader><Leader>fo`，所有的 `o` 字符都将高亮显示：

```
<cursor>L{a}rem ipsum d{b}l{c}r sit amet.
```

此时，按下 `b` 会跳到第二个 `o` ，如下：

```
Lorem ipsum d<cursor>olor sit amet.
```

例子演示完毕。

你可能会好奇为什么每次触发一个动作之前，为什么都需要按下两次`<Leader>`？

> 原来是作者考虑到默认的单个`<Leader>`可能与你安装的其他插件所使用的激活按键产生冲突，所以特意把默认的触发动作变成了`<Leader><Leader>`。

以上其实是官方给出的一个示例。主要演示了两个**EasyMotion**最基本的特性：单词移动动作`w`、查找移动动作`f`。

这两个动作很简单，但却非常实用，大部分的移动都可以靠这两个动作实现。

当然，如果你想要尝试更高级的技巧，可以参考 **EasyMotion** 的 **Github** 仓库官方文档：[**easymotion.txt**](https://github.com/easymotion/vim-easymotion/blob/master/doc/easymotion.txt)

### 最后

我的建议是，通过一次密集的突击训练，将`<Leader><Leader>`触发键，以及上面两个移动动作牢牢记住。

然后，再通过在日常生活工作中日复一日的大量使用，最终内化成肌肉记忆。

希望以上的内容能帮助你提升效率。


