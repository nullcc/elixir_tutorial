# 第二十一章 Erlang函数库

1.`binary`模块
2.格式化输出
3.`crypto`模块
4.`digraph`模块
5.Erlang持久化
6.`math`模块
7.`queue`模块
8.`rand`模块
9.`zip`和`zlib`模块

Elixir提供了非常优秀的和Erlang函数库的互操作性。事实上，Elixir不鼓励对Erlang函数库做简单的封装，它鼓励直接使用Erlang代码。在本节中我们将展示一些Erlang最常用的功能，这些功能Elixir中是没有的。

随着你越来越熟悉Elixir，你可能会想要探索[Erlang标准库](http://erlang.org/doc/apps/stdlib/index.html)。

## `binary`模块

Elixir内置的`String`模块处理UTF-8编码的二进制数据。在你处理不需要UTF-8编码的二进制数据时[`binary`模块](http://erlang.org/doc/man/binary.html)很有用。

```elixir
iex> String.to_charlist "Ø"
[216]
iex> :binary.bin_to_list "Ø"
[195, 152]
```

上面的例子展示了这两者的不同指之处；`String`模块返回Unicode码点，而`:binary`返回的是原始数据字节。

## 格式化输出

Elixir没有像C语言或其他语言的`printf`函数。幸运的是，Erlang标准库中有`:io.format/2`和`:io_lib.format/2`可以提供类似功能。前者在终端输出，后者会输出一个iolist。想要知道这两个函数和`printf`有什么不同，可以查阅[Erlang文档](http://erlang.org/doc/man/io.html#format-1)

```elixir
iex> :io.format("Pi is approximately given by:~10.3f~n", [:math.pi])
Pi is approximately given by:     3.142
:ok
iex> to_string :io_lib.format("Pi is approximately given by:~10.3f~n", [:math.pi])
"Pi is approximately given by:     3.142\n"
```

还需要注意Erlang的格式化函数需要特别注意对Unicode的处理。

## `crypto`模块

[`crypto`模块](http://erlang.org/doc/man/crypto.html)包含了哈希、数字签名、加密等函数：

```elixir
iex> Base.encode16(:crypto.hash(:sha256, "Elixir"))
"3315715A7A3AD57428298676C5AE465DADA38D951BDFAC9348A8A31E9C7401CB"
```

`:crypto`模块不是Erlang标准库的一部分，但它被包含在Erlang发行版中。这意味着当你使用它的时候你必须在你的项目中列出`:crypto`。你可以在`miz.exs`文件中包含它：

```elixir
def application do
  [extra_applications: [:crypto]]
end
```

## `digraph`模块

[`digraph`模块](http://erlang.org/doc/man/digraph.html)（也叫[digraph_utils](http://erlang.org/doc/man/digraph_utils.html)）包含处理顶点和变的有向图的函数。在构造图之后，这个模块中的算法将有助于发现两个顶点之间的最短路径，或是图中的循环。

给定三个顶点，找到第一个和最后一个顶点之间的最短路径。

```elixir
iex> digraph = :digraph.new()
iex> coords = [{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
iex> [v0, v1, v2] = (for c <- coords, do: :digraph.add_vertex(digraph, c))
iex> :digraph.add_edge(digraph, v0, v1)
iex> :digraph.add_edge(digraph, v1, v2)
iex> :digraph.get_short_path(digraph, v0, v2)
[{0.0, 0.0}, {1.0, 0.0}, {1.0, 1.0}]
```

请记住，`:digraph`中的函数变成了图结构，因为它们被实现为ETS表，这将在之后讨论。

## Erlang持久化

The modules ets and dets handle storage of large data structures in memory or on disk respectively.

ETS lets you create a table containing tuples. By default, ETS tables are protected, which means only the owner process may write to the table but any other process can read. ETS has some functionality to be used as a simple database, a key-value store or as a cache mechanism.

The functions in the ets module will modify the state of the table as a side-effect.

iex> table = :ets.new(:ets_test, [])
# Store as tuples with {name, population}
iex> :ets.insert(table, {"China", 1_374_000_000})
iex> :ets.insert(table, {"India", 1_284_000_000})
iex> :ets.insert(table, {"USA", 322_000_000})
iex> :ets.i(table)
<1   > {<<"India">>,1284000000}
<2   > {<<"USA">>,322000000}
<3   > {<<"China">>,1374000000}
The math module

The math module contains common mathematical operations covering trigonometry, exponential, and logarithmic functions.

iex> angle_45_deg = :math.pi() * 45.0 / 180.0
iex> :math.sin(angle_45_deg)
0.7071067811865475
iex> :math.exp(55.0)
7.694785265142018e23
iex> :math.log(7.694785265142018e23)
55.0
The queue module

The queue is a data structure that implements (double-ended) FIFO (first-in first-out) queues efficiently:

iex> q = :queue.new
iex> q = :queue.in("A", q)
iex> q = :queue.in("B", q)
iex> {value, q} = :queue.out(q)
iex> value
{:value, "A"}
iex> {value, q} = :queue.out(q)
iex> value
{:value, "B"}
iex> {value, q} = :queue.out(q)
iex> value
:empty
The rand module

rand has functions for returning random values and setting the random seed.

iex> :rand.uniform()
0.8175669086010815
iex> _ = :rand.seed(:exs1024, {123, 123534, 345345})
iex> :rand.uniform()
0.5820506340260994
iex> :rand.uniform(6)
6
The zip and zlib modules

The zip module lets you read and write ZIP files to and from disk or memory, as well as extracting file information.

This code counts the number of files in a ZIP file:

iex> :zip.foldl(fn _, _, _, acc -> acc + 1 end, 0, :binary.bin_to_list("file.zip"))
{:ok, 633}
The zlib module deals with data compression in zlib format, as found in the gzip command.

iex> song = "
...> Mary had a little lamb,
...> His fleece was white as snow,
...> And everywhere that Mary went,
...> The lamb was sure to go."
iex> compressed = :zlib.compress(song)
iex> byte_size song
110
iex> byte_size compressed
99
iex> :zlib.uncompress(compressed)
"\nMary had a little lamb,\nHis fleece was white as snow,\nAnd everywhere that Mary went,\nThe lamb was sure to go."
