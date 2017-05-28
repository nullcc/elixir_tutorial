# 类型规格和行为

1.类型和规格
  1.1函数规格
  1.2定义自己的类型
  1.3静态代码分析
2.行为
  2.1定义行为
  2.2采用行为

## 类型和规格

Elixir是一个动态类型的语言，所以Elixir的所有类型都会在运行时被推断出来。尽管如此，Elixir具有类型规格，它是一个符号，用于：

1.声明类型函数签名（规格）；
2.声明自定义数据类型。

### 函数规格

默认情况下，Elixir提供了一些基本类型，比如`integer`和`pid`，以及一些更加复杂的类型：比如，`round/1`函数，它把一个浮点数取整到最接近它的整数，它接受一个数作为参数（一个整型或一个浮点型），返回一个整型。正如你在它的[文档](https://hexdocs.pm/elixir/Kernel.html#round/1)中看到的，`round/1`函数的类型签名为：

```elixir
round(number) :: integer
```

`::`表示左边的函数返回一个右边类型的值。函数规格使用`@spec`指令声明，放在函数定义之前。`round/1`函数可以被写成：

```elixir
@spec round(number) :: integer
def round(number), do: # implementation...
```

Elixir还支持复合类型。举个例子，一个整型数列表的类型是`[integer]`。你可以在[规格文档](https://hexdocs.pm/elixir/typespecs.html)中看到所有Elixir的内置类型。

### 定义自己的类型

Elixir提供了很多有用的内置类型，在适当的时候定义自己的类型是非常方便的。这可以通过使用`@type`指令定义模块来实现。

假设我们有一个`LousyCalculator`模块，执行常用的算术运算（加、乘积等），但是它不返回数字，而是返回运算结果作为第一个元素，一个随机备注作为第二个参数的元组。

```elixir
defmodule LousyCalculator do
  @spec add(number, number) :: {number, String.t}
  def add(x, y), do: {x + y, "You need a calculator to do that?!"}

  @spec multiply(number, number) :: {number, String.t}
  def multiply(x, y), do: {x * y, "Jeez, come on!"}
end
```

正如你在上面例子中看到的，元组是复合类型，每个元组由内部类型标识。为了理解为什么写`String.t`而不是`String`，需要再看看[类型文档注释](https://hexdocs.pm/elixir/typespecs.html#notes)。

定义函数规格这种方式可以工作，但它很快会令人讨厌，因为我们在一遍又一遍地重复声明类型`{number, String.t}`。我们可以用`@type`指令来声明我们自己的类型。

```elixir
defmodule LousyCalculator do
  @typedoc """
  Just a number followed by a string.
  """
  @type number_with_remark :: {number, String.t}

  @spec add(number, number) :: number_with_remark
  def add(x, y), do: {x + y, "You need a calculator to do that?"}

  @spec multiply(number, number) :: number_with_remark
  def multiply(x, y), do: {x * y, "It is like addition on steroids."}
end
```

`@typedoc`指令和`@doc`和`@moduledoc`指令类似，用来文档化自定义类型。

可以在被定义模块外部导出用`@type`定义的自定义类型并使用它们：

```elixir
defmodule QuietCalculator do
  @spec add(number, number) :: number
  def add(x, y), do: make_quiet(LousyCalculator.add(x, y))

  @spec make_quiet(LousyCalculator.number_with_remark) :: number
  defp make_quiet({num, _remark}), do: num
end
```

如果你想让一个自定义类型变为私有的，你可以使用`@typep`指令替代`@type`指令。

### 静态代码分析

类型规格不仅可以作为开发者的附加文档。举个例子，Erlang分析工具使用类型规格来实现静态代码分析。这就是为什么在`QuietCalculator`的例子中，即使`make_quiet/1`是私有函数，我们还是为它声明函数规格。

## 行为

很多模块都共享相同的公共API。来看看[Plug](https://github.com/elixir-lang/plug)，它是web应用程序用可组合模块的规范。每个插件都是一个模块，它最少实现了两个公共函数：`init/1`和`call/2`。

行为提供了一种方式来：

* 定义一个必须由模块实现的函数集；
* 保证模块实现了函数集中的所有函数。

你可以把行为想像成面向对象语言如Java中的接口：一个函数签名集合，实现该接口的模块都必须实现这个函数集中的所有函数。

### 定义行为

假设我们要实现一些解析器，每个解析器解析一种结构化数据：举个例子，一个JSON解析器和一个YAML解析器。这两种解析器都以相同的方式工作：它们都提供一个`parse/1`函数和一个`extensions/0`函数。`parse/1`函数将返回一个Elixir表示的结构化数据，`extensions/0`函数则返回一个文件扩展名列表，可用于每个类型的数据（比如.json用于JSON文件）。

我们可以创建一个`Parser`行为：

```elixir
defmodule Parser do
  @callback parse(String.t) :: any
  @callback extensions() :: [String.t]
end
```

采用`Parser`行为的模块必须实现所有由`@callback`指令定义的函数。正如你所见，`@callback`指令需要一个函数名，而且需要一个像上面提到的`@spec`指令用到的函数规格。

### 采用行为

采用一个行为很简单：

```elixir
defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```

```elixir
defmodule YAMLParser do
  @behaviour Parser

  def parse(str), do: # ... parse YAML
  def extensions, do: ["yml"]
end
```

如果一个模块采用了一个给定的行为，但没有实现这个行为中被定义的方法，将产生一个编译期警告。
