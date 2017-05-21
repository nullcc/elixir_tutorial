# 第十四章 模块属性

1.作为注释
2.作为常量
3.作为临时存储

Elixir的模块属性有三个作用：

1.对模块进行注释，通常是说明用户或虚拟机使用模块的方式。
2.作为常量被使用。
3.作为编译期间的模块的临时存储。

让我们一个一个来看。

## 作为注释

Elixir从Erlang中吸收了模块属性的概念。举个例子：

```elixir
defmodule MyServer do
  @vsn 2
end
```

上面例子中，我们明确地设置了模块的版本号属性。Erlang虚拟机在代码重新载入时会使用`@vsn`来检查一个模块是够有更新。如果我们没有指定版本属性，则版本属性会被设置成模块函数的MD5值。

Elixir有少数保留的属性。以下是其中最常用的：

* `@moduledoc` - 提供当前模块的文档。
* `@doc` - 函数或宏的文档。
* `behaviour` - （请注意是英式拼写）用于指定一个OTP或用户自定义的行为。
* `before_compile` - 在模块被编译之前提供一个钩子。可以利用它在模块被编译之前注入函数到模块中。

`@moduledoc`和`@doc`是目前最常用的属性，我们推荐你多多使用它们。Elixir视文档为一等公民且提供了很多函数来访问文档。你可以通过[官方文档](https://hexdocs.pm/elixir/writing-documentation.html)来进一步了解Elixir中的文档。

让我们回头看看前几章中定义的`Math`模块，为它添加一些文档然后保存成`math.ex`文件：

```elixir
defmodule Math do
  @moduledoc """
  Provides math-related functions.

  ## Examples

      iex> Math.sum(1, 2)
      3

  """

  @doc """
  Calculates the sum of two numbers.
  """
  def sum(a, b), do: a + b
end
```

Elixir推荐使用Markdown来写可读性强的heredocs。Heredocs是多行字符串，它们被包裹在三引号中，它们能保持内部文本的格式。我们可以使用IEx来访问任何已编译模块的文档。

```elixir
$ elixirc math.ex
$ iex
iex> h Math # Access the docs for the module Math
...
iex> h Math.sum # Access the docs for the sum function
...
```

还提供了一个叫做`ExDoc`的工具来从文档生成HTML页面。

你可以查看[模块的文档](https://hexdocs.pm/elixir/Module.html)来了解模块支持的全部属性。Elixir还使用属性来定义[类型规格](http://elixir-lang.org/getting-started/typespecs-and-behaviours.html)

本节涵盖了内建的属性，然而，属性还可以被开发者或函数库用来支持自定义行为。

## 作为常量

Elixir开发者会经常把模块属性作为常量使用：

```elixir
defmodule MyServer do
  @initial_state %{host: "127.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```

> 注意：和Erlang不同，用户自定义的属性默认不会被存储在模块中。这些属性的值只在编译期间存在。开发者可以调用`Module.register_attribute/3`来让属性的行为和Erlang相似。

尝试访问一个未定义的属性会导致系统打印一个警告：

```elixir
defmodule MyServer do
  @unknown
end
warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it before access
```

最后，属性还能在函数内部被读取到：

```elixir
defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
```

每当在函数内部读取一个属性时，会对这个属性做一个快照。换句话说，属性的值是在编译期被读取的而不是在运行时。正如我们接下来要看到的，这种性质使得可以将模块属性作为编译期存储。

## 作为临时存储

Elixir生态圈中有一些[插件项目](https://github.com/elixir-lang/plug)，插件是Elixir中一种构建web函数库和框架的常用方式。

插件库还允许开发者定义他们自己的插件，这些插件可以运行在一个尾web服务器上：

```elixir
defmodule MyPlug do
  use Plug.Builder

  plug :set_header
  plug :send_ok

  def set_header(conn, _opts) do
    put_resp_header(conn, "x-header", "set")
  end

  def send_ok(conn, _opts) do
    send(conn, 200, "ok")
  end
end

IO.puts "Running MyPlug with Cowboy on http://localhost:4000"
Plug.Adapters.Cowboy.http MyPlug, []
```

上面例子中，我们使用`plug/1`宏来连接当一个web请求到来时需要调用的函数。在内部，每次你调用`plug/1`，插件库会把给定的参数存储在`@plugs`属性中。在模块被编译前，插件会运行一个回调函数，该函数定义了处理HTTP请求的函数(`call/2`)。这个函数会在`@plugs`中按顺序运行所有插件。

为了理解底层代码是如何运行的，我们需要宏，所以我们将在元编程指南中重新讨论这种模式。然而， 这里的重点是如何把模块属性作为存储来让开发者创建领域专属语言。

另一个例子来自[ExUnit框架](https://hexdocs.pm/ex_unit/)，它把模块属性当作注释和存储来使用：

```elixir
defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
```

ExUnit中的标签用来对测试用例进行注释。稍后会看到标签可以用于筛选测试用例。比如，虽然你可以在你的构建机器上运行外部测试用例子，但你可以选择不在你的机器上运行它们，因为外部测试用例运行起来很慢而且依赖于其他服务，

我们希望通过本节让你了解Elixir如何支持元编程，并且知道模块属性在其中的作用非常重要。

在接下来的章节中，我们在将探索异常处理和其他构建结构比如符号和解析之前讨论结构和协议。
