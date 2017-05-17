# 可枚举和流

1.可枚举
2.主动求值和惰性求值
3.管道操作符
4.流

## 可迭代对象

`Enum`模块用来处理Elixir中的可枚举的概念。我们已经学习了两种可枚举的类型：列表和映射。

```elixir
iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]
```

`Enum`模块提供了很多函数来处理可枚举类型的转换、排序、分组、过滤和检索元素。它是Elixir开发者经常使用的模块之一。

Elixir还提供范围：

```elixir
iex> Enum.map(1..3, fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.reduce(1..3, 0, &+/2)
6
```

`Enum`模块里的函数值针对数据结构中的枚举值。对于特定的操作，比如插入或更新一个特定的元素，你可能需要使用特定的模块来处理对应的数据类型。举个例子，如果你要在一个列表的指定位置上插入一个元素，则你必须使用`List`模块的`List.insert_at/3`函数， 每个数据类型都有适合它的操作，比如将一个值插入一个范围是没有意义的。

`Enum`模块中的函数是多态的，因为同一个函数可以处理多种数据类型。特别地，`Enum`模块中的函数可以用于任何实现了[可枚举协议](https://hexdocs.pm/elixir/Enumerable.html)的数据类型。我们将在后续章节中讨论协议；现在我们来看看特定类型的可枚举数据类型：流。

## 主动求值和惰性求值

`Enum`模块中的所有函数都是主动求值的。其中的很多函数都需要一个可枚举数据作为参数并返回一个列表：

```elixir
iex> odd? = &(rem(&1, 2) != 0)
#Function<6.80484245/1 in :erl_eval.expr/5>
iex> Enum.filter(1..3, odd?)
[1, 3]
```

这意味着当使用`Enum`执行多个操作时，每个操作都会产生中间结果列表直到获得最后结果：

```elixir
iex> 1..100_000 |> Enum.map(&(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
7500000000
```

上面的例子使用了管道操作。我们以一个数字范围列表开始，依次对这个列表中的每个元素乘以3。这个操作会创建并返回一个100_000个元素的列表。然后我们保留列表中的所有奇数，返回一个50_000元素的列表，然后求和这个列表。

## 管道操作符

上面代码片段中的`|>`符号是管道操作符：它左侧表达式的结果作为右侧表达式第一个输入参数。这和Unix的`|`操作符很类似，其目的是突出数据在一系列函数中的转换过程。对比不使用`|>`操作符的代码，使用`|>`操作符的代码显得非常简洁：

```elixir
iex> Enum.sum(Enum.filter(Enum.map(1..100_000, &(&1 * 3)), odd?))
7500000000
```

查看[管道操作符](https://hexdocs.pm/elixir/Kernel.html#%7C%3E/2)的文档以获取更多信息。

## 流

作为`Enum`的替代品，Elixir提供了[Stream模块](https://hexdocs.pm/elixir/Stream.html)来支持惰性操作：

```elixir
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
7500000000
```

流是惰性、可组合的枚举接口。

上面的例子中，`1..100_000 |> Stream.map(&(&1 * 3))`返回一个流，表示在范围`1..100_000`上的映射计算。

```elixir
iex> 1..100_000 |> Stream.map(&(&1 * 3))
#Stream<[enum: 1..100000, funs: [#Function<34.16982430/1 in Stream.map/2>]]>
```

此外，流是可组合的因为我们可以管道化流操作：

```elixir
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
#Stream<[enum: 1..100000, funs: [...]]>
```

和生成中间结果列表不同的是，流构建一系列只在调用底层`Enum`模块时才会被最终执行的计算。流很适合处理大量数据，甚至是数据量无限的集合。

`Stream`模块中的许多函数接受任何可枚举的数据作为参数，并返回一个流作为结果。`Stream`模块还提供了创建流的函数。比如，`Stream.cycle/1`可以创建一个拥有指定个数元素的循环流。需要小心的是，不要在这样一个流上调用`Enum.map/2`，这将导致无限循环：

```elixir
iex> stream = Stream.cycle([1, 2, 3])
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
```

另一方面，`Stream.unfold/2`可以用来从指定的初始值生成数据。

```elixir
iex> stream = Stream.unfold("hełło", &String.next_codepoint/1)
#Function<39.75994740/2 in Stream.unfold/2>
iex> Enum.take(stream, 3)
["h", "e", "ł"]
```

另一个有趣的函数是`Stream.resource/3`，它可以用来包装资源，保证资源在被枚举前打开，并在使用完后关闭，即使过程中出现错误。举个例子，我们可以用它来流化一个文件：

```elixir
iex> stream = File.stream!("path/to/file")
#Function<18.16982430/2 in Stream.resource/3>
iex> Enum.take(stream, 10)
```

上面这个例子将获取指定文件的前10行。这表明流在处理大文件甚至是慢速资源例如网络资源的时候非常有用。

一次性讲完`Enum`模块和`Stream`模块的所有功能是很困难的，不过你可以慢慢学习它们。特别地，比较好的学习方式是先学习和使用`Enum`模块，直到在一些需要惰性求值特殊场景下才使用`Stream`模块，比如处理大量数据，甚至是数据量无限的集合。

接下来我们将学习Elixir的核心：进程，进程让我们能够以一种容易理解的方式编写并发的、并行的和分布式的程序。
