# 模块和函数

1.编译
2.脚本模式
3.具名函数
4.函数捕获
5.默认参数

在Elixir中我们把一些函数组合成模块。在之前的几章中我们已经使用过很多不同的模块了，比如[`String`模块](https://hexdocs.pm/elixir/String.html)：

```elixir
iex> String.length("hello")
5
```

我们可以使用`defmodule`宏来创建自定义的模块。使用`def`宏来在模块中定义函数：

```elixir
iex> defmodule Math do
...>   def sum(a, b) do
...>     a + b
...>   end
...> end

iex> Math.sum(1, 2)
3
```

在后面的章节中，我们的例子会越来越长，要把它们输入到shell中会比较困难。是时候来学习一下如何编译Elixir的代码和运行Elixir脚本了。

## 编译

大多数情况下，把模块写进文件对于编译和重用模块会很方便。假设我们有一个文件叫做`math.ex`，内容如下：

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end
```

可以使用`elixirc`来编译文件：

```elixir
$ elixirc math.ex
```

执行这个命令会生成一个叫做`Elixir.Math.beam`的文件，其中包含所定义模块的字节码。如果我们再次启动`iex`，就可以直接使用我们的模块（字节码文件和`iex`在相同目录下）：

```elixir
iex> Math.sum(1, 2)
3
```

Elixir项目通常被组织成三个目录：

* ebin - 包含被编译成的字节码
* lib - 包含elixir源代码（一般是.ex文件）
* test - 包含测试文件（一般是.exs文件）

在真实项目开发中，`mix`这个构建工具将为你编译代码和设置合适的路径。出于学习的目的，Elixir还支持脚本模式，脚本模式更灵活，且不会产生任何编译输出文件。

## 脚本模式

除了文件扩展名`.ex`，Elixir还支持`.exs`文件作为脚本文件名。除了使用意图上，Elixir对待这两种文件没什么区别。`.ex`文件用于编译，`.exs`文件用于脚本。当被执行时，尽管只有`.ex`文件会被编译成字节码并以`.beam`的形式写入磁盘，但两种扩展名的文件都会被编译并加载它们的模块到内存中。

例如，我们创建一个文件`math.exs`：

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end

IO.puts Math.sum(1, 2)
```

用以下命令执行它：

```elixir
$ elixir math.exs
```

这个文件将在内存中被编译和执行，打印结果"3"。执行过程中不会创建字节码。在下面的例子中，你最好自己把代码写到脚本文件中并像上面那样执行它。

## 具名函数

在一个模块内，我们可以使用`def/2`和`defp/2`定义函数和私有函数。用`def/2`定义的函数可以被其他模块调用，但私有函数只能在本模块内部使用。

```elixir
defmodule Math do
  def sum(a, b) do
    do_sum(a, b)
  end

  defp do_sum(a, b) do
    a + b
  end
end

IO.puts Math.sum(1, 2)    #=> 3
IO.puts Math.do_sum(1, 2) #=> ** (UndefinedFunctionError)
```

函数声明也支持哨兵和多重字句。如果一个函数有多个子句，Elixir会尝试每个子句直到找到匹配的子句。下面是一个实现判断给定的数字是否为0的函数：

```elixir
defmodule Math do
  def zero?(0) do
    true
  end

  def zero?(x) when is_integer(x) do
    false
  end
end

IO.puts Math.zero?(0)         #=> true
IO.puts Math.zero?(1)         #=> false
IO.puts Math.zero?([1, 2, 3]) #=> ** (FunctionClauseError)
IO.puts Math.zero?(0.0)       #=> ** (FunctionClauseError)
``

给定的参数无法匹配任何子句时将抛出错误。

和`if`结构类似，具名函数支持`do:`和`do/end`块语法，即我们之前学到的`do/end`这种便捷的关键字列表语法。比如，我们可以编辑`math.exs`文件：

```elixir
defmodule Math do
  def zero?(0), do: true
  def zero?(x) when is_integer(x), do: false
end
```

这两种写法的代码行为是相同的。你可以在单行代码的情况下用`do`而在多行代码的情况下用`do/end`。

## 函数捕获

在整个教程中，我们都使用符号`函数名/元数`来作为函数参考。巧的是，可以使用这个符号来获取一个具名函数。启动`iex`，像下面这样运行`math.exs`文件：

```elixir
$ iex math.exs
iex> Math.zero?(0)
true
iex> fun = &Math.zero?/1
&Math.zero?/1
iex> is_function(fun)
true
iex> fun.(0)
true
```

请记住Elixir区分匿名函数和具名函数，匿名函数在调用时必须在函数变量名和左括号之间加上一个`.`。捕获操作符通过这种方式使得具名函数可以被赋值给一个变量、调用和传递匿名函数

本地函数或被导入的函数，像`is_function/1`，可以在不需要模块的情况下被捕获：

```elixir
iex> &is_function/1
&:erlang.is_function/1
iex> (&is_function/1).(fun)
true
```

注意捕获语法也可以作为一种创建函数的快捷方式：

```elixir
iex> fun = &(&1 + 1)
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> fun.(1)
2
```

`&1`表示被传递给函数的第一个参数。上面的`&(&1+1)`实际上等同于`fn x -> x + 1 end`。上面这种语法是创建函数的快捷方式。

如果你想捕获一个模块中的函数，你可以用`&Module.function()`：

```elixir
iex> fun = &List.flatten(&1, &2)
&List.flatten/2
iex> fun.([1, [[2], 3]], [4, 5])
[1, 2, 3, 4, 5]
```

`&List.flatten(&1, &2)`等同于`fn(list, tail) -> List.flatten(list, tail)`，在这种情况下，等同于`&List.flatten/2`。在[Kernel.SpecialForms](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#&/1)中你可以找到更多关于捕获操作符的信息。

## 默认参数

具名函数还支持默认参数：

```elixir
defmodule Concat do
  def join(a, b, sep \\ " ") do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

默认值可以是任何表达式，但它在函数定义的时候不会被求值。每次函数被调用并使用默认参数值的时候，默认参数值的表达式会被求值：

```elixir
defmodule DefaultTest do
  def dowork(x \\ "hello") do
    x
  end
end
iex> DefaultTest.dowork
"hello"
iex> DefaultTest.dowork 123
123
iex> DefaultTest.dowork
"hello"
```

当一个拥有默认参数值的函数使用多个子句，就需要创建一个函数头（没有函数体）来声明默认值：

```elixir
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ")

  def join(a, b, _sep) when is_nil(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
IO.puts Concat.join("Hello")               #=> Hello
```

当使用默认参数值时，必须小心避免函数定义覆盖。考虑下面的例子：

```elixir
defmodule Concat do
  def join(a, b) do
    IO.puts "***First join"
    a <> b
  end

  def join(a, b, sep \\ " ") do
    IO.puts "***Second join"
    a <> sep <> b
  end
end
```

如果把上面的代码保存为`"concat.ex"`并编译它，Elixir会提示以下警告：

```elixir
警告：这个子句无法被匹配，因为第2行的子句总是被匹配。
```

编译器告诉我们当使用两个参数调用`join`函数时，总是会选择第一个定义执行，只有在传递三个参数的情况下才会执行第二个定义。

```elixir
$ iex concat.exs
iex> Concat.join "Hello", "world"
***First join
"Helloworld"
iex> Concat.join "Hello", "world", "_"
***Second join
"Hello_world"
```

到这里就结束了我们对模块的简要介绍了。在后面的章节中，我们将学习如何用具名函数实现递归、能从其他模块中导入函数的指令以及模块属性。
