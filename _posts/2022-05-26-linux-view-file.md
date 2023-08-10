---
layout: post
title: 【Linux】查看文件内容的5个常用命令
subtitle: 掌握基础命令
tags: [linux]
---

目录
- [前言](#前言)
- [查看文件内容最常用的 5 个命令](#查看文件内容最常用的-5-个命令)
  - [cat](#cat)
  - [nl](#nl)
  - [less](#less)
  - [tail](#tail)
  - [head](#head)
- [最后](#最后)

### 前言

不管是在日常工作连接远程服务器中，还是在平时个人电脑使用中（如果使用的`Mac OS` 或 `Linux`系统的话），都离不开强大的`Terminal`终端。

比如，查看远程服务器上的程序运行日志，使用终端查看各种文件等等。

在这篇文章中简单总结下在`Terminal`终端中，如何快速查看文件内容的5个常用命令。

--- 
### 查看文件内容最常用的 5 个命令

#### cat

`cat`命令应该是在`Linux`中查看文件内容最常见的命令了。

使用`cat`命令会打印指定文件的所有内容到标准输出上，比如你的屏幕。

`cat`命令最简单的用法，是直接在cat`命令`后面跟上文件即可。

如下图：

![cat](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/cat.png)

#### nl

`nl`命令跟`cat`命令很相识，它的不同之处在于每一行的前面多了行号的显示。

如下图：

![nl](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/nl.png)

#### less

`less`命令一次只会显示一个页面的文件内容。

可以通过 `j`、`k` 两个按键进行上、下浏览文件内容，使用 `q` 可以随时退出。

如下图：

![less](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/less.gif)

上图中使用 `less` 命令打开 `testFile.txt`文件，然后通过 `j`、`k` 按键上、下浏览文件内容，最后，使用 `q` 键退出。

#### tail

`tail` 命令用于查看文件内容的最后一部分，默认显示的行数是10行。

当然，如果你想让 `tail` 命令显示更多的文件内容，可以使用 `-n number` 这个参数，`number` 代表行数；

如下动态图：

![tail](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/tail.gif)

#### head

`head` 命令跟`tail`很相识，只不过它们查看的文件内容的方向是相反的。

`head` 命令用于查看文件内容的前面部分，默认显示的行数也是10行。

当然，如果想显示更多的行数的话，也是可以使用 `-n number` 这个参数，`number` 代表行数；

如下动态图：

![head](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/head.gif)

--- 
### 最后

以上就是在`Linux`上常用的查看文件内容的 5 个命令啦！

简单却很实用的命令！

不过，以上都只是基础的玩法，`Linux`的各种命令都有着各种各样的参数可以满足你的各种需求，使用`man`命令去探索上面5个命令更高阶的用法吧！
