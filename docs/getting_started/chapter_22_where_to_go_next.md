# 第二十二章 接下来的学习路径

1.构建你的第一个Elixir项目
2.元编程
3.社区和其他资源
4.Erlang简介

渴望学习更多内容吗？请继续阅读！

## 构建你的第一个Elixir项目

为了让你的第一个应用跑起来，Elixir内置了一个叫做Mix的构建工具。你可以使用下面的命令来开始你的新项目：

```elixir
$ mix new path/to/new/project
```

我们已经写好了一个指南，涵盖了如何构建一个Elixir应用，它有自己的监控树、配置、测试等。这个应用是一个分布式的键-值存储，我们将键-值对放到桶中，并将这些桶分布到多个节点上：

* [Mix和OTP](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html)

## 元编程

由于其元编程的支持，Elixir是可扩展的、具有高可定制性的编程语言。Elixir中大多数元编程都使用宏来实现，宏在很多场合非常有用，特别是用来写领域专属语言。我们写了一个介绍宏背后的基本机制的指南，展示了如何写宏，以及如何使用宏来创建领域专属语言：

* [Elixir中的元编程](https://elixir-lang.org/getting-started/meta/quote-and-unquote.html)

## 社区和其他资源

这里有一[节](https://elixir-lang.org/learning.html)来介绍我们推荐的学习Elixir和探索它的生态系统的书籍、视频和其他资源。外面也有大量的Elixir资源，比如演讲、开源项目和其他社区提供的学习材料。

Don’t forget that you can also check the source code of Elixir itself, which is mostly written in Elixir (mainly the lib directory), or explore Elixir’s documentation.

别忘了你还可以直接查阅[Elixir的源代码](https://github.com/elixir-lang/elixir)，Elixir的代码大部分用Elixir写成（大部分在lib目录下），或者查阅[Elixir的文档](https://elixir-lang.org/docs.html)。

## Erlang简介

Elixir运行在Erlang虚拟机上，而且一个Elixir开发者早晚都会面对Erlang函数库。这里列出了一个涵盖Erlang基本原理和高级特性的在线资源列表：

* [Erlang Syntax: A Crash Course](https://elixir-lang.org/crash-course.html)简要介绍了Erlang的语法。每个代码片段都有等价的Elixir代码片段与之对应。这让你不仅可以接触到Erlang的语法，还可以回顾一些在本教程中学到的知识。

* Erlang官网有一个简短的[入门教程](http://www.erlang.org/course/concurrent_programming.html)，里面配有图片，简要地描述了Erlang并发编程的原语。

* [Learn You Some Erlang for Great Good!](http://learnyousomeerlang.com/)是一个非常优秀的Erlang介绍，它包含设计原则、标准库、最佳实践等内容。一单你通读了上面提到的速成教程，你可以安全地跳过本书的前几章的语法介绍。当你开始学习[The Hitchhiker’s Guide to Concurrency](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency)这章时，你会发现真正有意思的事情才刚刚开始。
