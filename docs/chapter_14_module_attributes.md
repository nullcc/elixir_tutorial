# 第十四章 模块属性

1.作为注释
2.作为常量
3，作为临时存储

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

Elixir developers will often use module attributes as constants:

defmodule MyServer do
  @initial_state %{host: "127.0.0.1", port: 3456}
  IO.inspect @initial_state
end
Note: Unlike Erlang, user defined attributes are not stored in the module by default. The value exists only during compilation time. A developer can configure an attribute to behave closer to Erlang by calling Module.register_attribute/3.
Trying to access an attribute that was not defined will print a warning:

defmodule MyServer do
  @unknown
end
warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it before access
Finally, attributes can also be read inside functions:

defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
Every time an attribute is read inside a function, a snapshot of its current value is taken. In other words, the value is read at compilation time and not at runtime. As we are going to see, this also makes attributes useful to be used as storage during module compilation.

As temporary storage

One of the projects in the Elixir organization is the Plug project, which is meant to be a common foundation for building web libraries and frameworks in Elixir.

The Plug library also allows developers to define their own plugs which can be run in a web server:

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
In the example above, we have used the plug/1 macro to connect functions that will be invoked when there is a web request. Internally, every time you call plug/1, the Plug library stores the given argument in a @plugs attribute. Just before the module is compiled, Plug runs a callback that defines a function (call/2) which handles HTTP requests. This function will run all plugs inside @plugs in order.

In order to understand the underlying code, we’d need macros, so we will revisit this pattern in the meta-programming guide. However the focus here is on how using module attributes as storage allows developers to create DSLs.

Another example comes from the ExUnit framework which uses module attributes as annotation and storage:

defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
Tags in ExUnit are used to annotate tests. Tags can be later used to filter tests. For example, you can avoid running external tests on your machine because they are slow and dependent on other services, while they can still be enabled in your build system.

We hope this section shines some light on how Elixir supports meta-programming and how module attributes play an important role when doing so.

In the next chapters we’ll explore structs and protocols before moving to exception handling and other constructs like sigils and comprehensions.
