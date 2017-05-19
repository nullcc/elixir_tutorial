# 第十二章 I/O和文件系统

1.`I/O`模块
2.`File`模块
3.`Path`模块
4.进程和进程组组长
5.`iodata`和`chardata`

本章将快速介绍输入/输出机制和文件系统相关任务，以及相关的模块，比如`IO`模块、`File`模块和`Path`模块。

我们在入门指南的早期就已经提到本章的一些内容了。然而，我们注意到IO系统能引导我们对Elixir和VM的了解和点燃对它们的好奇心。

`IO`模块

`IO`模块是Elixir中读写标准输入/输出（`:stdio`）、标准错误输出、文件和其他IO设备的主要机制。它的使用非常简单：

```elixir
iex> IO.puts "hello world"
hello world
:ok
iex> IO.gets "yes or no? "
yes or no? yes
"yes\n"
```

默认情况下，`IO`模块中的函数从标准输入中读取数据，并把结果输出到标准输出中。我们可以通过传递参数来改变输出方式，比如，传入`:stderr`作为参数（这样做是为了输出到标准错误设备上）：

```elixir
iex> IO.puts :stderr, "hello world"
hello world
:ok
```

## `File`模块

`File`模块包含了打开文件和IO设备的一些函数。默认情况下，文件会以二进制模式被打开，开发者可以使用`IO`模块的`IO.binread/2`和`IO.binwrite/2`函数：

```elixir
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
iex> IO.binwrite file, "world"
:ok
iex> File.close file
:ok
iex> File.read "hello"
{:ok, "world"}
```

一个文件还可以以`:utf8`编码被打开，`File`模块会以UTF-8字节编码模式从文件读取和解析字节数据。

除了打开文件、读取文件和写入文件的函数，`File`模块还有很多操作文件系统的函数。那些函数都以UNIX等价命令命名。比如，`File.rm/1`函数用来删除文件，`File.mkdir/1`函数用来创建目录，`File.mkdir_p/1`函数用来创建目录和它的父路径链。`File.cp_r/2`和`File.rm_rf/1`函数分别用来复制和删除文件和目录（即也复制和删除目录中的内容）。

你还会注意到`File`模块中的函数有两个变种：一个是常规的，另一个结尾有一个感叹号（!）。比如，上面的例子在读取`"hello"`文件时，我们使用`File.read/1`，我们也可以使用`File.read!/1`：

```elixir
iex> File.read "hello"
{:ok, "world"}
iex> File.read! "hello"
"world"
iex> File.read "unknown"
{:error, :enoent}
iex> File.read! "unknown"
** (File.Error) could not read file "unknown": no such file or directory
```

带有`!`的版本会返回文件内容而不是一个元组，而且如果有错误发生，会抛出一个异常。

当你想用模式匹配处理不同情况得到不同结果时，推荐使用不带`!`的版本的函数：

```elixir
case File.read(file) do
  {:ok, body}      -> # do something with the `body`
  {:error, reason} -> # handle the error caused by `reason`
end
```

然而，如果你期望文件是存在的，爆炸方法会更有用，因为当文件不存在时它会抛出一个有意义的错误信息。请避免写下面这种代码：

```elixir
{:ok, body} = File.read(file)
```

当发生错误时，`File.read/1`会返回`{:error, reason}`，上面代码中的模式匹配会失败。这种情况下你仍然会得到想要的结果（一个错误），但这个信息和你给定的模式并不匹配（因此，就会引发一个不好排查的错误。）

因此，如果你不想处理错误输出信息，推荐你使用`File.read!/1`。

## `Path`模块

大多数`File`模块中的函数都需要一个文件路径作为参数。最常见的情况是，那些路径代表的是常规的二进制文件。`Path`模块提供了处理这种路径的功能：

```elixir
iex> Path.join("foo", "bar")
"foo/bar"
iex> Path.expand("~/hello")
"/Users/jose/hello"
```

相比于直接操作路径字符串，更推荐使用`Path`模块里的函数来处理和路径有关的字符串，因为`Path`模块能兼容不同操作系统。最后，请记住在Windows下执行和文件相关的操作时，Elixir会自动把斜杠（`/`）转换成反斜杠（`\`）。

到这里，我们已经涵盖了Elixir中处理IO和文件系统的主要模块。在下一小节中，我们将讨论一些关于IO的高级话题。这些东西在编写Elixir代码时不是必须要掌握的，所以你大可以跳过它们，不过这些知识对我们了解VM中IO系统的实现很有启发性。

## 进程和进程组组长

你可能会注意到函数`File.open/2`返回一个类似`{:ok, pid}`的元组：

```elixir
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
```

上面的代码工作正常是因为`IO`模块实际上工作在进程之上（参看第十一章）。当你写`IO.write(pid, binary)`时，`IO`模块会向指定pid的进程发送一个消息执行所需操作。来看看如果我们向进程发送这个消息会发生什么：

```elixir
iex> pid = spawn fn ->
...>  receive do: (msg -> IO.inspect msg)
...> end
#PID<0.57.0>
iex> IO.write(pid, "hello")
{:io_request, #PID<0.41.0>, #Reference<0.0.8.91>,
 {:put_chars, :unicode, "hello"}}
** (ErlangError) erlang error: :terminated
```

在调用`IO.write/2`之后，我们看到`IO`模块发送的请求（一个四元元组）被打印了出来。然后我们发现失败了，因为我们没有提供`IO`模块预期的某种结果。

`StringIO`模块基于字符串提供了IO设备消息的一个实现：

```elixir
iex> {:ok, pid} = StringIO.open("hello")
{:ok, #PID<0.43.0>}
iex> IO.read(pid, 2)
"he"
```

通过使用进程对IO设备进行建模，Erlang虚拟机允许为了节点之间读/写文件，在同个网络中的不同节点间交换文件进程。在所有IO设备中，每个进程都有一个特殊的东西：组长。

当你向`:stdio`写数据时，你实际上是在向它的组长发送一个消息，它是一个标准输出的文件描述符：

```elixir
iex> IO.puts :stdio, "hello"
hello
:ok
iex> IO.puts Process.group_leader, "hello"
hello
:ok
```

每个进程可以配置进程组长，并在不同的情况下使用。比如，当在一个远程终端上执行代码时，它保证远程终端节点上的信息被重定向输出到触发请求的终端中。

## `iodata`和`chardata`

在上面的所有例子中，我们在写文件时都使用二进制模式。在[“二进制、字符串和字符列表”](./chapter_6_binaries_strings_and_char_lists.md)中，我们提到了字符串是如何由字节组成的，而字符列表是如何由unicode码点组成的。

`IO`模块和`File`模块中的函数也允许用列表作为参数进行调用。不仅如此，它们还允许列表、整型和二进制的混合列表作为参数：

```elixir
iex> IO.puts 'hello world'
hello world
:ok
iex> IO.puts ['hello', ?\s, "world"]
hello world
:ok
```

然而，在IO操作中使用列表需要注意一些问题。一个列表可能会以一些字节或一些字符组成，具体使用哪种编码形式取决于具体的IO设备。如果打开某个文件不需要指定编码，那么这个文件就是以原始模式被打开的，那么就应该使用`IO`模块中那些以`bin*`打头的函数来处理。那些函数需要以一个`iodata`作为参数；也就是说，它们需要一个整型字节列表和二进制数据作为参数。

另一方面，使用`IO`模块中的其他函数来以`:utf8`编码模式来操作`:stdio`和文件。那些函数需要一个`char_data`作为参数，即一个字符列表或字符串。

虽然这里有一些微妙的差别，但你也只需在想传递列表到这些函数时才需要要考虑这些问题。二进制文件由底层字节来表示，所以它们的表示方式永远是“raw”。

到这里就结束了IO设备和IO相关功能的介绍。我们已经学习了四个Elixir模块了：`IO`模块、`File`模块、`Path`模块和`StringIO`模块，以及虚拟机如何使用进程作为IO机制的底层实现和如何使用`chardata`和`iodata`来处理IO操作。
