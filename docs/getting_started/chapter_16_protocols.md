# 第十六章 协议

1.协议和结构体
2.实现`Any`
  2.1推导
  2.2回退到`Any`
3.内置协议
4.协议加固

协议是Elixir中实现多态的一种机制。只要实现了特定的协议，任何数据类型都可以工作于该协议之上。让我们看一个例子。

在Elixir中，有两种查看数据结构中有多少元素的方式：`length`和`size`。`length`意味着该数据结构的长度必须被计算出来。一个例子是`length(list)`需要遍历整个可列表来计算它的长度。另一方面，`tuple_size(tuple)`和`byte_size(binary)`则不需要动态计算大小，因为它们的大小信息是被预先计算出来存放在数据结构中的。

即使我们已经拥有了内置的针对特定数据结构的获取数据结构大小的函数（例如`tuple_size/1`），但我们也可以为所有需要预先计算大小的数据结构定义一个通用的`Size`协议，让它们实现这个协议。

可以像下面这样定义协议：

```elixir
defprotocol Size do
  @doc "Calculates the size (and not the length!) of a data structure"
  def size(data)
end
```

`Size`协议需要那些实现它的数据类型实现一个叫做`size`的函数，这个函数接受一个参数（我们想要知道它的大小的数据结构）。现在我们可以让所有需要兼容size方法的数据结构实现这个协议：

```elixir
defimpl Size, for: BitString do
  def size(string), do: byte_size(string)
end

defimpl Size, for: Map do
  def size(map), do: map_size(map)
end

defimpl Size, for: Tuple do
  def size(tuple), do: tuple_size(tuple)
end
```

我们无须为列表实现`Size`协议，因为列表没有预先计算“size”信息的必要，列表的长度需要被实时计算（使用`length/1`）。

现在有`Size`协议的定义和实现在手，我们来开始使用它：

```elixir
iex> Size.size("foo")
3
iex> Size.size({:ok, "hello"})
2
iex> Size.size(%{label: "some label"})
1
```

传入一个没有实现该协议的数据类型到协议方法中会抛出一个异常：

```elixir
iex> Size.size([1, 2, 3])
** (Protocol.UndefinedError) protocol Size not implemented for [1, 2, 3]
```

可以对所有Elixir的数据类型实现协议：

* Atom
* BitString
* Float
* Function
* Integer
* List
* Map
* PID
* Port
* Reference
* Tuple

## 协议和结构体

当协议和结构体一起使用时，Elixir的可扩展性才真正能发挥威力。

在之前的章节，我们已经了解了虽然结构体本质上是映射，但它并不共享映射的协议实现。比如，`MapSet`（基于映射的集合）被实现成结构体。让我们来试试用`MapSet`实现`Size`协议：

```elixir
iex> Size.size(%{})
0
iex> set = %MapSet{} = MapSet.new
#MapSet<[]>
iex> Size.size(set)
** (Protocol.UndefinedError) protocol Size not implemented for #MapSet<[]>
```

结构体不会共享映射的协议实现，它需要有自己的协议实现。由于一个`MapSet`可以使用`MapSet.size/1`获取它预先被计算出来的大小信息，我们可以为让它实现`Size`协议：

```elixir
defimpl Size, for: MapSet do
  def size(set), do: MapSet.size(set)
end
```

如果需要的化，你可以实现你自定义结构的大小的定义。不仅如此，你还可以用结构体构建更具鲁棒性的数据类型，比如队列，并为它们实现相关协议，例如`Enumerable`和`Size`。

```elixir
defmodule User do
  defstruct [:name, :age]
end

defimpl Size, for: User do
  def size(_user), do: 2
end
```

## 为`Any`实现协议

手动实现所有类型的协议非常麻烦，很快你就会厌倦这种繁琐枯燥的工作。在这种情况下，Elixir提供了两种选择：我们可以明确地为我们的数据类型实现协议，或者自动让所有类型实现协议。这两种情况我们都需要为`Any`实现协议。

### 推导

Elixir允许我们基于`Any`的实现来继承协议实现。我们先为`Any`实现如下协议：

```elixir
defimpl Size, for: Any do
  def size(_), do: 0
end
```

上面的实现可以说是不合理的。举个例子，说`PID`和`Integer`的大小为0毫无意义。

然而，我们不应该直接使用`Any`的实现，为了使用这个实现，我们需要明确地告诉结构体要推导出`Size`协议：

```elixir
defmodule OtherUser do
  @derive [Size]
  defstruct [:name, :age]
end
```

当使用推导时，Elixir将基于`Any`的实现来为结构体`OtherUser`实现`Size`协议。

### 回退到`Any`

`@derive`的一种替代方案是明确地告诉协议当找不到实现时回退到`Any`。这可以通过在协议定义中设置`@fallback_to_any`为`true`来实现：

```elixir
defprotocol Size do
  @fallback_to_any true
  def size(data)
end
```

正如我们在前面章节所述，`Any`的`Size`实现是无法适用于所有数据类型的。这就是为什么`@fallback_to_any`是一个可选的行为的原因。对于大多数协议来说，当协议未被实现时抛出一个异常是比较合适的做法。也就是说，假设我们在上一节中实现了`Any`的协议：

```elixir
defimpl Size, for: Any do
  def size(_), do: 0
end
```

现在那些没有实现`Size`协议的所有数据类型（包括结构体）将被认为size为`0`。

推导和回退到`Any`哪种方式比较好取决于具体场景，但相比于隐式，Elixir开发者更喜欢明确地说明，你可能会看到很多函数库在开头使用`@derive`。

## 内置协议

Elixir有一些内置协议。在前面的章节中，我们讨论了`Enum`模块，它提供了很多函数来让那些实现了`Enumerable`协议的数据结构使用：

```elixir
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2, 4, 6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

另一个有用的例子是`String.Chars`协议，它指定了如何把一个具有字符结构数据转换成字符串。这通过`to_string`函数暴露出来：

```elixir
iex> to_string :hello
"hello"
```

请注意Elixir中的字符串插值会调用`to_string`函数：

```elixir
iex> "age: #{25}"
"age: 25"
```

上面这个代码段能工作是因为数字实现了`String.Chars`协议。下面的例子传入一个元组，会导致一个错误：

```elixir
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当需要“打印”一个复杂的数据结构时，可以使用`inspect`函数，这个函数基于`Inspect`协议：

```elixir
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

`Inspect`协议用来把任意数据结构转换成可读的文本形式。这很像IEx的打印功能：

```elixir
iex> {1, 2, 3}
{1, 2, 3}
iex> %User{}
%User{name: "john", age: 27}
```

请记住，依照惯例，每当被检视的值以`#`开头，它表示这个数据结构不是一个合法的Elixir语法。这意味着检视协议在这个数据结构上是不可逆的，因为有可能在转换过程中丢失信息：

```elixir
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```

Elixir中还有一些其他的协议，但这里只涵盖最通用的一些协议。

## 协议加固

当使用Mix构建工具开发Elixir项目时，你会看到如下一些输出：

```elixir
Consolidated String.Chars
Consolidated Collectable
Consolidated List.Chars
Consolidated IEx.Info
Consolidated Enumerable
Consolidated Inspect
```

这些是Elixir携带的所有协议，它们正在被加固。因为一个协议可以被任何数据结构使用，协议必须检查给定类型是否实现了该协议。这种操作可能开销很大。

然而，当你使用类似Mix这种构建工具编译项目之后，我们知道所有模块都已经被定义了，包括协议和它们的实现。这样的话，该协议可以被加固成一个非常简单和快速的模块调用。

从Elixir v1.2开始，协议加固会自动发生在所有项目中。我们将在《Mix和OTP指南》中构建我们自己的项目。
