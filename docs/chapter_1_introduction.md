# Elixir 简介

* 安装
* 交互模式
* 运行脚本
* 提问

欢迎！

在这个入门教程中你将学习Elixir的基础知识：语法、如何定义模块、如何操纵常用的数据结构等。本章将关注Elixir的安装，并且让你成功运行Elixir的交互shell：iex。

我们需要准备的东西有：

Elixir - 版本号>=1.4.0
Erlang - 版本号>=18.0
让我们开始吧！

> 如果你在这个入门教程或本网站上发现任何错误，请发送一个错误报告或一个pull request到我们的问题跟踪列表上。

## 安装

如果你还没安装Elixir，请到[安装页面](http://elixir-lang.org/install.html)。一旦完成安装，可以运行 elixir --version 来查看Elixir的当前版本号。

## 交互模式

当安装了Elixir后，你将获得三个可执行程序：iex、elixir和elixirc。如果你从源码编译Elixir或者使用一个打包的版本，你可以在bin目录下找到这三个程序。

现在，让我们从运行iex(Windows下是iex.bat)开始接触Elixir交互模式，我们可以在其中键入任何Elixir表达式来获取相应的结果。来我们来看几个基本的表达式练练手。

Open up iex and type the following expressions:

打开iex，输入以下表达式：

```elixir
Erlang/OTP 19 [erts-8.1] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

Interactive Elixir (1.4.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> 40 + 2
42
iex(2)> "hello" <> " world"
"hello world"
```

请注意一些细节，比如版本号可能和你在你的机器上显示有些许不同，这并无大碍。从现在开始我们在iex中只需要关注代码。需要离开iex可以键入Ctrl+C两次。

看来我们已经准备好继续往前了！在下一章中，我们会比较多地使用到Elixir的交互模式来熟悉这门语言的构造和基本使用方式。

> 注意：如果你在Windows环境下，使用iex.bat --werl命令可以让iex根据你使用的命令行终端来优化使用体验。

## 运行脚本

熟悉了一些Elixir的基本内容后你可能会想要尝试写一点简单的程序。我们可以尝试把以下Elixir代码保存到一个文件中：

```elixir
IO.puts "Hello world from Elixir"
```

将文件保存为simple.exs，然后用elixir执行它：

```shell
$ elixir simple.exs
Hello world from Elixir
```

之后我们会学习如何编译Elixir代码(在第八章中)和如何使用Mix构建工具(在Mix和OPT指南中)。现在，让我们开始学习第二章。

## 提问

在学习本入门教程时，遇到问题是很正常的事情，毕竟提问题就是学习过程的一部分！这里有很多你可以通过提问来进一步学习Elixir的地方：

* [elixir-lang on freenode IRC](irc://irc.freenode.net/elixir-lang)
* [Elixir on Slack](https://elixir-slackin.herokuapp.com/)
* [Elixir Forum](http://elixirforum.com/)
* [elixir tag on StackOverflow](https://stackoverflow.com/questions/tagged/elixir)

当你提问时，请记住两个技巧：

请用提问“在Elixir中如何解决某问题”来替代“在Elixir中如何做到某件事”。换句话说，不要具体去问如何实现一个特定的解决方案，相反你应该描述清楚目前遇到的问题。正确说明问题可以给其他人带来更多的上下文信息和更少的你个人的主观意见，这对解决问题有利。

如果需要提问，请在你的提问中包含尽可能多的信息，比如：你的Elixir版本号、代码片段和栈跟踪的错误信息。可以使用类似Gist的站点来粘贴这些信息。
