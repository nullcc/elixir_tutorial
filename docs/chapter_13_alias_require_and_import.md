# 第十三章 alias，require和import

1.alias
2.require
3.import
4.use
5.理解别名
6.模块嵌套
7.联合使用alias/import/require/use

为了实现代码重用，Elixir提供了三个指令（`alias`，`require`和`import`）和一个宏`use`，使用总结如下：

```elixir
# 为一个模块起别名，下面的例子中可以使用Bar代替Foo.Bar
alias Foo.Bar, as: Bar

# 包含模块来使用其中的宏
require Foo

# 从Foo模块中导入函数，这样就可以在没有前缀的`Foo.`的情况下调用它们
import Foo

# 调用Foo模块中的自定义代码作为扩展点
use Foo
```

下面我们来详细地研究它们。请记住前三个叫做指令，因为它们有词法作用域，而`use`是一个常见的扩展点。

## alias（别名）

`alias`允许你对任何模块设置一到多个别名。

假设一个模块使用了`Math.List`模块中的一个特定实现的列表。`alias`指令允许在这个模块定义中使用`List`来代替`Math.List`：

```elixir
defmodule Stats do
  alias Math.List, as: List
  # In the remaining module definition List expands to Math.List.
end
```

在`Stats`模块中还是可以用完全限定名称`Elixir.List`来访问原始的列表。

>注意：Elixir中的所有模块都被定义在一个主Elixir命名空间中。然而，为了使用方便，你可以在引用它们的时候省略“Elixir.”前缀。

别名经常被用于定义短名称。实际上，调用`alias`但不使用`:as`可选参数会自动把模块的最后一部分名称作为别名，比如：

```elixir
alias Math.List
```

和下面的效果是一样的：

```elixir
alias Math.List, as: List
```

注意`alias`有词法作用域，这可以让你在特定的函数内部设置别名，而不影响到外部词法作用域：

```elixir
defmodule Math do
  def plus(a, b) do
    alias Math.List
    # ...
  end

  def minus(a, b) do
    # ...
  end
end
```

上面这个例子中，我们在`plus/2`函数内部定义了别名，这个别名只会在`plus/2`内部生效，不会影响`minus/2`和其他所有函数。

## require

Elixir提供了宏这种机制来支持元编程（生成代码的代码）。宏会在编译期被展开。

模块里的公共函数是全局可用的，但为了使用它们，你需要引入它们所在的模块。

```elixir
iex> Integer.is_odd(3)
** (UndefinedFunctionError) function Integer.is_odd/1 is undefined or private. However there is a macro with the same name and arity. Be sure to require Integer if you intend to invoke this macro
iex> require Integer
Integer
iex> Integer.is_odd(3)
true
```

在Elixir中，`Integer.is_odd/1`被定义成一个宏，所以它可以被当成一个哨兵来使用。这意味着为了调用`Integer.is_odd/1`，我们需要引入`Integer`模块。

请注意像`alias`、`require`这些指令，都是有词法作用域的。我们将在后续章节讨论更多关于它们的问题。

## import

无论何时当想要不使用完全限定名称来轻松地访问其他模块中的函数或宏时，我们使用`import`。例如，如果我们想多次使用`List`模块中的`duplicate/2`函数，我们可以先导入它：

```elixir
iex> import List, only: [duplicate: 2]
List
iex> duplicate :ok, 3
[:ok, :ok, :ok]
```

在这个例子中，我们只从`List`模块中导入了duplicate（元数为2）函数。虽然`:only`是可选的，但使用它可以避免在当前命名空间中导入指定模块的所有函数，我们推荐这么做。`:except`选项则可以导入指定模块中除了给定的列表中的函数以外的所有函数。

`import`还支持对`:only`选项指定`:macros`和`:functions`。比如，想导入所有宏，可以这么写：

```elixir
import Integer, only: :macros
```

或者想导入所有函数，可以这么写：

```elixir
import Integer, only: :functions
```

请注意导入也有词法作用域。这意味着我们可以在函数内部导入特定的宏或函数而不影响外部词法作用域：

```elixir
defmodule Math do
  def some_function do
    import List, only: [duplicate: 2]
    duplicate(:ok, 10)
  end
end
```

上面例子中，被导入的`List.duplicate/2`只在特定的函数中可见。`duplicate/2`在该模块的其他函数中或其他模块中不可用。

请注意，导入一个模块会自动引用这个模块。

## use

`use`宏经常被开发者用于扩展当前词法作用域，常见于模块。

比如，为了使用ExUnit框架来编写测试用例，开发者需要use`ExUnit.Case`模块：

```elixir
defmodule AssertionTest do
  use ExUnit.Case, async: true

  test "always pass" do
    assert true
  end
end
```

在这段代码背后，`use`包含了指定的模块，然后在指定模块上调用`__using__/1`回调把这个模块的一些代码注入到当前上下文中。一般来说，以下模块：

```elixir
defmodule Example do
  use Feature, option: :value
end
```

会被编译成：

```elixir
defmodule Example do
  require Feature
  Feature.__using__(option: :value)
end
```

## 理解别名

现在你可能会觉得奇怪：Elixir中的别名到底是什么，它是如何代表原始名称的？

Elixir中的别名是一个首字母大写的标识符（和`String`、`Keyword`等类似），会在编译期间被转换为一个原子。比如，`String`别名默认被转换成原子`:"Elicir.String"`：

```elixir
iex> is_atom(String)
true
iex> to_string(String)
"Elixir.String"
iex> :"Elixir.String" == String
true
```

通过使用`alias/2`指令，我们会改变别名被展开时的形式。

在Erlang虚拟机中别名被展开为原子，因此在Elixir中也是。模块总是用原子来表示。比如，下面是我们用来调用Erlang模块的机制：

```elixir
iex> :lists.flatten([1, [2], 3])
[1, 2, 3]
```

## 模块嵌套

既然我们已经讨论了别名，那我们就来看看Elixir中的嵌套和它是如何工作的。考虑以下例子：

```elixir
defmodule Foo do
  defmodule Bar do
  end
end
```

上面例子将定义两个模块：`Foo`和`Foo.Bar`。只要在同一个词法作用域内，第二个模块可以在`Foo`模块内部以`Bar`被访问。上面代码等同于：

```elixir
defmodule Elixir.Foo do
  defmodule Elixir.Foo.Bar do
  end
  alias Elixir.Foo.Bar, as: Bar
end
```

如果以后`Bar`模块的定义被从`Foo`模块中移出，则它必须被以全称（`Foo.Bar`）或一个用`alias`指定定义的别名来引用。

>注意：在Elixir中，我们不需要在定义`Foo.Bar`模块之前定义`Foo`模块，因为Elixir会把所有模块名称转换称原子。你可以在不定义任何中间模块的情况下定义任意的嵌套模块。（即定义`Foo.Bar.Baz`模块而不定义`Foo`模块和`Foo.Bar`模块）

在之后的章节中我们将看到，别名在宏中扮演了重要的角色，它将保证宏是简洁的。

## 联合使用alias/import/require/use

从Elixir v1.2开始，支持同时`alias`、`import`或`require`多个模块。这在我们定义嵌套模块时非常有用，这在构建Elixir应用程序时很常见。比如，假设你有一个应用，所有模块都被嵌套定义在`MyApp`模块下，你可以用下面的代码同时为`MyApp.Foo`、`MyApp.Bar`和`MyApp.Baz`设置别名：

```elixir
alias MyApp.{Foo, Bar, Baz}
```

到这里我们就结束了Elixir模块的介绍。接下来的一章将介绍模块熟悉。
