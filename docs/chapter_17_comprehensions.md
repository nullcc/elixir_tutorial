# 第十七章 解析

1.生成器和过滤器
2.位串生成器
3.`:into`选项

在Elixir中，一种常见的操作是迭代一个可枚举对象，过滤其元素和把元素映射到另一个列表中。解析是这种操作的语法糖：它们以特殊的形式来处理那些通用的操作。

比如，我们可以把一个列表中的整型映射成它的平方：

```elixir
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```

解析由三个部分构成：生成器、过滤器和可迭代对象。

## 生成器和过滤器

上面的表达式中，`n <- [1, 2, 3, 4]`是生成器。它负责生成在解析过程中要用到的值。任何可枚举对象都可以被作为右值当成生成器表达式：

```elixir
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```

生成器表达式也支持左值的模式匹配；所有不匹配的值会被忽略。设想一种场景，我们有一个键为原子`:good`和`:bad`的关键字列表，我们只想计算键为`:good`的值的平方：

```elixir
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

一种模式匹配的可替代方案是，过滤器可以被用来过滤出一些特定的元素。比如，我们可以挑选出3的倍数，忽略其他的值：

```elixir
iex> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
iex> for n <- 0..5, multiple_of_3?.(n), do: n * n
[0, 9]
```

解析会丢弃所有过滤器返回`false`或`nil`的元素；保留其余的元素。

相比于使用`Enum`模块和`Stream`模块中的等价函数，解析提供了一种更加简洁的使用方式。此外，解析还可以接受多个生成器和过滤器。下面的例子展示了接受一个目录列表，计算每个目录中每个文件的大小：

```elixir
dirs = ['/home/mikey', '/home/james']
for dir  <- dirs,
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  File.stat!(path).size
end
```

在计算两个列表的笛卡尔积时也用到了多个生成器：

```elixir
iex> for i <- [:a, :b, :c], j <- [1, 2], do:  {i, j}
[a: 1, a: 2, b: 1, b: 2, c: 1, c: 2]
```

一个更高级的多生成器和多过滤器的例子是毕达哥拉斯三元组。一个毕达哥拉斯三元组是一个正整型数集合，集合中的数满足`a*a + b*b = c*c`，我们写一个解析来处理这个问题，保存在文件`triple.exs`中：

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n,
        b <- 1..n,
        c <- 1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

在命令行执行：

```elixir
iex triple.exs
```

```elixir
iex> Triple.pythagorean(5)
[]
iex> Triple.pythagorean(12)
[{3, 4, 5}, {4, 3, 5}]
iex> Triple.pythagorean(48)
[{3, 4, 5}, {4, 3, 5}, {5, 12, 13}, {6, 8, 10}, {8, 6, 10}, {8, 15, 17},
 {9, 12, 15}, {12, 5, 13}, {12, 9, 15}, {12, 16, 20}, {15, 8, 17}, {16, 12, 20}]
```

当搜索范围很大时，上述代码的开销很昂贵。此外，由于元组`{b, a, c}`和元组`{a, b, c}`在做为毕达哥拉斯三元组时是完全相同的。我们可以通过重构生成器部分的代码来优化这个解析并且消除重复的结果，比如：

```elixir
defmodule Triple do
  def pythagorean(n) when n > 0 do
    for a <- 1..n-2,
        b <- a+1..n-1,
        c <- b+1..n,
        a + b + c <= n,
        a*a + b*b == c*c,
        do: {a, b, c}
  end
end
```

最后，请记住，在解析的生成器、过滤器和块中被赋值的变量，不会对解析外部造成影响。

## 位串生成器

解析还支持使用位串生成器，当你需要解析位串流的时候非常有用。下面的例子接受一个二进制表示的像素列表，并将它们转换成红、绿、蓝三原色的元组：

```elixir
iex> pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>
iex> for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
[{213, 45, 132}, {64, 76, 32}, {76, 0, 0}, {234, 32, 15}]
```

位串生成器可以和“常规的”可枚举生成器混合使用，它也支持过滤器。

## `:into`选项

上面的例子中，所有解析都返回列表作为结果。然而，可以通过传入`:into`选项给解析来把解析的返回结果改成其他数据结构。

举个例子，一个位串生成器可以使用`:into`选项来方便地移除一个字符串内的所有空白：

```elixir
iex> for <<c <- " hello world ">>, c != ?\s, into: "", do: <<c>>
"helloworld"
```

集合、映射和其他字典类型也可以使用`:into`选项。一般来说，只要实现了`Collectable`协议的数据结构都可以使用`:into`选项。

`:into`的一个常见用法是在不使用映射中的键的情况下转换映射中的值：

```elixir
iex> for {key, val} <- %{"a" => 1, "b" => 2}, into: %{}, do: {key, val * val}
%{"a" => 1, "b" => 4}
```

我们来看看另一个使用流的例子。`IO`模块提供了流（流是`Enumerable`且`Collectable`的），可以使用解析来实现一个echo服务器，它回返回用户输入的大写版本：

```elixir
iex> stream = IO.stream(:stdio, :line)
iex> for line <- stream, into: stream do
...>   String.upcase(line) <> "\n"
...> end
```

现在在终端中输入任何字符串，你会看到相同的信息以大写形式打印出来。不幸的是，这个例子会导致你的IEx卡住，所以你需要按两次`Ctrl+C`来退出。:)
