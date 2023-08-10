---
layout: post
title: 【Mac OS】在Windows与Mac OS之间快捷传输文件
subtitle: 
tags: [macos, windows]
---

{: .box-success}
当你的个人电脑从Windows系统转向Mac OS系统，并且还不时需要在两种系统之间传输文件时，以下的方法将会帮助你。

目录
- [前言](#前言)
- [Windows端](#windows端)
- [Mac端](#mac端)
- [`SMB`是什么？](#smb是什么)
- [参考](#参考)

### 前言
本文中的方法本质是通过在同一网络下的计算机之间建立共享文件夹的方式完成文件传输，使用SMB通讯协议。


### Windows端

1、新建文件夹；

2、右键文件夹—>属性—>共享—>高级共享，勾选“共享此文件夹”，设置“权限”，按需设置读写权限。

3、获取本地IP地址，`Win+R`打开`cmd`窗口，输入命令`ipconfig`即可。

### Mac端

1、打开 Finder（访达）

2、找到顶部菜单栏中的 Go（前往），选择底部的 Connect to Server（连接服务器）

3、输入 Windows 端的 IP 地址，并在前面加上前缀：smb:// ，如下示例：
> smb://192.168.2.16

4、点击 Connect （连接）即可。

经过上面两步的设置，以后你就可以在你的Windows与Mac之间随意传输文件了，只需要将文件拖拉到共享文件夹中即可。

### `SMB`是什么？

服务器消息块 (Server Message Block，简称SMB) 是一种通信协议。

### 参考

1、[**wikipedia 页面**](https://en.wikipedia.org/wiki/Server_Message_Block)

2、[**Server Message Block protocol (SMB protocol)**](https://www.techtarget.com/searchnetworking/definition/Server-Message-Block-Protocol)