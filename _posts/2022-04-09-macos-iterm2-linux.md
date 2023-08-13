---
layout: post
title: 【Linux】iTerm2中配置SSH连接Linux服务器的快捷方法
subtitle: 快速连接remote server！ 
tags: [linux, tool, iterm]
---

目录
- [前言](#前言)
- [步骤](#步骤)
  - [1.在`.ssh/`目录下创建配置文件](#1在ssh目录下创建配置文件)
  - [2.在配置文件中增加命令](#2在配置文件中增加命令)
  - [3.配置`iTerm2`的`profile`](#3配置iterm2的profile)
  - [4.尽情“无脑”使用吧](#4尽情无脑使用吧)
- [结语](#结语)

## 前言

平时在Mac上使用`iTerm2`登录远程服务器时，总是需要手动输入`ssh`命令以及密码，比较麻烦。

再加上如果平时有多个远程服务器在使用，就意味着需要记住多组`host`地址和密码，就更麻烦了。

于是，便在网上找了找有没有什么方法可以让这个过程不那么麻烦，变得简单点，可以“无脑”操作，甚至完全自动化。

经过一番搜索，整理如下。

## 步骤

### 1.在`.ssh/`目录下创建配置文件

```shell
# 打开.ssh目录
cd ~/.ssh/
# 使用vim创建并进入文件，也可使用其他创建文件命令，如touch
# 文件名建议设置成描述用途的单词
vim XXX   
```

### 2.在配置文件中增加命令

在文件里面输入如下命令

 ```shell
#!/usr/bin/expect -f

#文件一定要以 #!/usr/bin/expect -f 开头
#设置ip地址、用户名、端口号、密码等
set host XXXX
set user XXXX
set port XXXX
set password XXXX
set timeout -1

#一系列自动化登录的命令。
spawn ssh $user@$host
expect "*password:*"
send "$password\r"
interact
expect eof
 ```

简单解释下上面的一些命令。

- `set host XXXX`
  > 表示设置一个名为`host`，值为XXXX的**变量**。可以使用`$`取到该变量的值。

- `spawn ssh $user@$host`
  >  表示使用`spawn`打开`ssh`这个程序，并取前面设置好的用户名和主机地址作为参数。<br>
  >  `spawn`这个命令可以用来运yy行任何交互式脚本或程序，非常强大。

- `expect "*password:*"`
  > `expect`命令表示期待读取到什么字符。后面字符中的`*`表示通配符。用过`ssh`的小伙伴应该都知道，当输入`ssh`之后，会要求你要输入密码。

- `send "$password\r"`
  > `send`命令表示发送。这里是将预设置的`password`变量发送到程序中。

- `interact`
  > 这个命令用来执行交互动作。

### 3.配置`iTerm2`的`profile`

打开`iTerm2`的`Preferences`，根据以下序号配置，如下图
![iterm-preferences](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/iterm-preferences.jpeg)
步骤说明：

1. 选择`profiles`

2. 点击`+`号按钮会出现`New Profile`

3. 填上想要设置的`Name`名称

4. 在下拉框中选择`Command`

5. 在右边的框框中填写配置文件的位置。

> 每一个`profile`都可以看作一个模版，里面可配置你想要在打开`iTerm`时执行的任何命令。上图中前面带⭐️名为`Defaul`的`profile`，其实就是一个默认的模版。

### 4.尽情“无脑”使用吧

所有的配置完毕，回到`iTerm2`的主界面。

如上图，第一步是点击`iTerm2`菜单栏中的`Profiles`，会出现下拉选项，显示出所有你已经配置的`profile`。第二步，选中你想要使用的对应的`profile`就OK了。
![iterm-profiles](https://raw.githubusercontent.com/MaoPingZou/img_repo/master/blog/iterm-profiles.jpeg)

---
## 结语
现在，终于不用每次使用`iTerm2`登录远程服务器时，都需要使用`ssh`命令输入主机地址和密码了。

世界，变得更简单了那么一点！

也许，还有更简单的方法，或者有好用的软件，如果你知道，欢迎邮箱留言推荐给我！