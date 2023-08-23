---
layout: post
title: 【Vim】多窗口分割编辑
subtitle: 不止一个窗口
tags: [vim, tool]
---

在 `Vim` 中，可以使用 `split` 和 `vsplit` 命令来实现多窗口分割编辑。

具体使用方式:

- 横向分割窗口:
命令：`split filename`
解释：横向分割出一个新窗口,并打开`filename`文件
- 纵向分割窗口:
命令：`vsplit filename`
解释：纵向分割出一个新窗口,并打开 `filename`文件
- 切换窗口:
  方式一：`ctrl + w + w` 
  解释：循环切换窗口

  方式二：`ctrl + w + h/j/k/l`
  解释：切换到左、下、上、右窗口

- 关闭窗口:
命令：`:q` 或 `:close` 

通过这些命令，可以很方便地在 `Vim` 中管理多个分割窗口，从而实现多文件并排编辑、查看等功能。

掌握好多窗口编辑可以大幅提高工作效率。
