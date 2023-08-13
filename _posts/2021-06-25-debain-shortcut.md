---
layout: post
title: 【Debain】Debian系统设置常用工具的键盘快捷键
subtitle: 多用快捷键
tags: [debain, tool]
---

{: .box-success}
无论使用什么系统，使用快捷键都能提高我们使用系统都效率。下面是在`Debian`系统中，为一些常用的工具设置快捷键，提高生产力。

目录
- [最基本步骤](#最基本步骤)
- [为工具设置快捷键](#为工具设置快捷键)
  - [为命令行工具`Terminal`设置快捷键](#为命令行工具terminal设置快捷键)
  - [为截图工具`Flameshot`设置快捷键](#为截图工具flameshot设置快捷键)
- [最后](#最后)

## 最基本步骤
- 打开 **系统设置**
- 选择 **设备**，再选择 **键盘**
- 下拉到最下面可以看到一个 **+**，点击弹出弹窗，如下
![0-debain](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/0-debain.png)

- 最重要的一步就是找到对应工具的 `command` 命令了。在终端上输入 **`dpkg -l | grep xxxx`** 即可获取。<br>
例如以下：2 -> 代表的就是对应工具的 `command`
![1-debain](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/1-debain.png)

这样就开始为各种各样的工具设置快捷键啦！

## 为工具设置快捷键

### 为命令行工具`Terminal`设置快捷键
`Terminal`是平常最常用到一个工具，所以，为这个工具设置键盘快捷键当然是非常有必要的了！

步骤：
- 输入名称 **`Terminal`**，输入命令 **`gnome-terminal`**，
- 点击设置快捷键，此时按下想要设置的快捷键键位组合，我这里使用的键位是 **`Ctrl + Alt  + T`**
- 完成

### 为截图工具`Flameshot`设置快捷键
`Flameshot`是一个很好用的截图工具。但也许只有为它设置了快捷键后，你才会真正喜欢上它的强大！

步骤：
- 输入名称 **`Flameshot`**，输入命令 **`flameshot gui`**，
- 点击设置快捷键，此时按下想要设置的快捷键键位组合，我这里使用的键位是 **`Ctrl + M`**
- 完成

## 最后
继续你愉快的`Debian`系统学习之旅吧！