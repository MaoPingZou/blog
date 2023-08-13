---
layout: post
title: 【Git】IDEA的Shelve与Git的Stash之间的区别
subtitle: 了解了区别才知道如何更好使用
tags: [git, idea]
---

目录
- [前言](#前言)
- [IDEA的`Shelve`功能](#idea的shelve功能)
  - [`Shelve`是一个什么样的功能？](#shelve是一个什么样的功能)
  - [`Shelve`功能有哪些优点？](#shelve功能有哪些优点)
  - [如何使用`Shelve`功能？](#如何使用shelve功能)
  - [如何使用`Unshelve`功能？](#如何使用unshelve功能)
  - [有哪些需要注意的地方？](#有哪些需要注意的地方)
- [Git的`Stash`功能](#git的stash功能)
  - [`Stash`是一个什么样的功能？](#stash是一个什么样的功能)
  - [`Stash`有什么优点？](#stash有什么优点)
  - [如何使用`Stash`？](#如何使用stash)
  - [如何使用`Unstash`功能。](#如何使用unstash功能)
  - [有哪些需要注意的地方？](#有哪些需要注意的地方-1)
- [总结一下两者的区别](#总结一下两者的区别)
- [最后](#最后)

## 前言

在日常开发中，会遇到需要在不同的开发任务之间切换的情况，此时，如果手中有尚未写完的代码，就需要使用某种方法先保存起来，之后再继续处理没写完的代码。

有两种方法可以解决上述问题：一个是利用IDEA提供的`Shelve`功能，另一个是利用Git提供的`Stash`功能。

如果你也是一个使用IDEA作为集成开发环境的话，而且使用的版本控制管理工具也是Git的话，那这篇文章也许对你在日常工作中遇到上面的情况时有所帮助。

本文中介绍了IDEA的`Shelve`与Git的`Stash`这两个功能的特点、如何使用、需要注意的地方。

## IDEA的`Shelve`功能

### `Shelve`是一个什么样的功能？

> `Shelve`是IDEA本身提供的一个功能，用来**暂时保存尚未`commit`的修改**。

注意，这里为什么要特别指出“**尚未`commit`**”的修改？

因为IDEA提供的`Shelve`这个功能只能作用于被版本控制管理的文件。

也就是说，如果你的文件，没有被类似Git、Subversion等版本控制软件管理的话，是无法使用`Shelve`这个功能的。

### `Shelve`功能有哪些优点？

> 1.既可以暂存单个文件，也可以暂存整个`ChangeList`中所有文件。
>
> 2.可以根据需要多次应用被`Shelve`的修改。

### 如何使用`Shelve`功能？

> 1. 在`Local Changes`中找到尚未`commit`的文件。
>
> 2. 右键点击想要`Shelve`的文件或整个`ChangeList`。
> 3. 选择`Shelve Changes...`。
> 4. 会弹出对话框，填写`Commit Message`一栏（它会作为此次创建的shelf的名字）。
> 5. 然后点击`Shelve Changes`按钮。
> 6. 之后，会自动转到`Shelf`一栏（出现在`Local Changes`右边），里面放着所有被`Shelve`的文件。

因IDEA在不断迭代更新，有些地方可能进行了优化导致位置可能发生变化。

所以以上给出的是一个比较通用的方法：直接从`Local Changes`中找到需要`Shelve`的文件。

### 如何使用`Unshelve`功能？

既然文件的修改被`Shelve`起来了，肯定还是希望能在某一时刻能恢复回来继续工作。

所以，也就有了对应的`Unshelve`功能。

大致的步骤如下：

在`Shelf`（`Local Changes`旁边）页面中，找到需要`Unshelve`的文件，右键单击在上下文菜单中找到`Unshelve...`选项，单击后在对话框中填写相关信息，然后点击对话框右下角`Unshelve Changes`即可。

### 有哪些需要注意的地方？

> 在`Unshelve`时，如果当前`Local Changes`中某个文件已有的修改与`Shelve`中对应文件的修改位置相同，则会产生冲突。

## Git的`Stash`功能

### `Stash`是一个什么样的功能？

> `Stash`是Git提供的一个功能，会将**所有尚未`commit`的修改进行储存**。

注意，Stash操作是会将**所有**尚未`commit`的文件都一并保存的。

### `Stash`有什么优点？

> 1. 可以将`Stash`的内容应用到一个已存在的分支或者以`Stash`为基础创建一个新的分支。
>
> 2. `Stash`的内容可以根据需要多次应用到所需的任何分支上。

### 如何使用`Stash`？

> 1. 在`Local Changes`中找到尚未`commit`的文件。
>
> 2. 右键点击任意一个文件，出现上下文菜单，将光标移动到Git上。
>
> 3. 右边出现Git的相关选项，点击`Stash Changes...`选项。
>
> 4. 会弹出对话框，在`Message`框中填写关于此次`stash`内容的说明。
>
> 5. 最后，点击`Create Stash`按钮即可。

这里也是选择了比较通用的方式找到需要被`stash`的文件，而不需要关心IDEA主菜单上关于Git的变化。

### 如何使用`Unstash`功能。

与`Shelve`功能有个对应的`Unshelve`功能一样，`Stash`功能也有一个对应的叫`UnStash`的功能。

但`UnStash`并没有一个像存储着`Shelve`文件的`Shelf`页面，而是需要在菜单栏的Git中找。

使用的步骤大致如下：

> 在IDEA主菜单上，找到`Git | Uncommitted Changes | Unstash Changes`选项；
>
> 出现对话框，选择想要应用`stash`的分支；
>
> 再从`stash`列表中选择需要应用的`stash`；
>
> 最后点击`Apply Stash`按钮即可。

### 有哪些需要注意的地方？

> 1. 在一系列的`commit`之后再恢复`stash`时，可能会导致冲突，需要手动解决。
>
> 2. 恢复`stash`时，当前的工作区不能有尚未`commit`的修改。

## 总结一下两者的区别

1. `Stash`是Git提供的功能，而`shevle`是IDEA本身提供的功能。
2. `Stash`只能针对当前整个分支所有未`commit`的文件进行操作；而`Shelve`更灵活，可以对单个或多个未`commit`的文件进行操作；
3. 要恢复`stash`的文件时（`unstash`操作），如果本地有尚未`commit`的修改，那么此次`unstash`操作会失败。IDEA右下角会弹出提醒：你的本地修改将会被`merge`覆盖。请`commit`，`stash`或`revert`它们之后再继续。而恢复`shelve`的文件时（`unshelve`操作），无论本地有没有被修改的文件都不会影响。解决可能产生的冲突即可。
4. `Stash`的文件是放在`.git`文件中；`Shelve`的文件是放在`.idea/shelf`文件中。这意味着`Stash`的文件比`Shelve`的文件有更强的可移植性。

## 最后

本篇文章的目的是为了对IDEA的`Shelve`和Git的`Stash`两个功能进行一个大致的比较，并讲解最基础的使用方法。

因此，本篇文章并没有讲解在使用`Shelve`或`Unshelve`、`Stash`或`Unstash`时弹出的对话框中更多的具体选项该如何选择使用，如果希望更好的掌握这两个功能，可进一步查阅资料了解。


<br>
本文参考链接：

- JetBrains: [Shelve and unshelve changes](https://www.jetbrains.com/help/idea/2021.3/shelving-and-unshelving-changes.html)
- JetBrains: [Use Git to work on several features simultaneously](https://www.jetbrains.com/help/idea/2021.3/work-on-several-features-simultaneously.html)
- StackOverflow: [Git Stash vs Shelve in IntelliJ IDEA/Netbeans](https://stackoverflow.com/questions/28008139/git-stash-vs-shelve-in-intellij-idea)