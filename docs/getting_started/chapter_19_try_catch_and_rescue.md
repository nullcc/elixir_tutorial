# 第十九章 try,catch和rescue

1.Errors
2.Throws
3.Exits
4.After
5.Else
6.变量作用域

Elixir有三种错误机制：errors、throws和exits。在本章汇中我们将谈挨个探讨它们，并介绍它们的使用场景。

## Errors

错误（或异常）被用在当代码中有异常行为的时候。一个会产生错误的例子是尝试在一个原子上加上一个数：

```elixir
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

可以在任何时候使用`raise/1`来抛出一个运行时错误：

```elixir
iex> raise "oops"
** (RuntimeError) oops
```

可以使用错误名和一个关键字列表作为参数调用`raise/2`来抛出其他异常：

```elixir
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

你还可以通过创建一个模块，并在内部使用`defexception`来定义你自己的错误；通过这种方式，你将创建一个和它所处模块同名的错误。最常见的场景是用一个消息定义一个自定义异常：

```elixir
iex> defmodule MyError do
iex>   defexception message: "default message"
iex> end
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

可以使用`try/rescue`结构来捕获并处理错误：

```elixir
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "oops"}
```

上面的例子捕获了运行时错误并返回错误本身，然后打印在`iex`会话中。

如果错误对你来说没有用，你就不需要返回它：

```elixir
iex> try do
...>   raise "oops"
...> rescue
...>   RuntimeError -> "Error!"
...> end
"Error!"
```

然而，在实践中，Elixir开发者很少使用`try/rescue`结构。举个例子，很多语言会强制让你在无法打开一个文件的时候捕获并处理一个错误。Elixir则提供了一个`File.read/1`函数，它返回一个包含了是否成功打开文件的标志的元组：

```elixir
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```

这里没有使用`try/rescue`。当你想处理打开一个文件的多种可能结果时，你可以用模式匹配来解决：

```elixir
iex> case File.read "hello" do
...>   {:ok, body}      -> IO.puts "Success: #{body}"
...>   {:error, reason} -> IO.puts "Error: #{reason}"
...> end
```

当然，是否在打开一个文件出现问题的时候抛出异常完全取决你的应用程序。这就是为什么Elixir不会强制让你在使用`File.read/1`和其他函数的时候使用异常。相反，这让开发者可以自行选择一个最佳的处理方式。

在你十分确信某个文件存在的情况下（并且缺少该文件是一个错误），你可以使用`File.read!/1`：

```elixir
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:305: File.read!/1
```

标准库中的许多函数都遵循一种模式，这些函数的返回值是一个元组，可以用于模式匹配，但它们有一个对应的变体，这个变体不会返回一个元组，而是在错误时抛出一个异常。惯例是创建一个函数（`foo`），它返回`{:ok, result}`或者`{:error, reason}`元组，另一个和它对用的函数（`foo!`，和`foo`同名但末尾有一个`!`），它接受和`foo`相同的参数，但它会在发生错误时抛出一个错误。`foo!`会在正常执行时返回结果（不包含在一个元组中）。`File`模块是这种惯例的很好的例子。

在Elixir中，我们避免使用`try/rescue`是因为我们不使用错误来控制代码流程。我们从字面上来理解错误：它们是用来处理非预期和/或异常情况的。如果你确实需要流程控制结构，则应使用`throws`。我们接下来将看到它。

## Throws

在Elixir中，一个值可以被抛出，在稍后被捕获。`throw`和`catch`被用在那些不可能获取到值，除非使用`throw`和`catch`的情况下。

这些情况在实践中很少见，除非是库的接口没有提供一个合适的API的情况下。举个例子，假设`Enum`模块没有提供任何查找一个值的API，这时我们需要在一个列表中找到第一个是13的倍数的数字：

```elixir
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

由于`Enum`模块没有提供合适的API，在实践中可以使用`Enum.find/2`：

```elixir
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## Exits

所有的Elixir代码都运行在相互通信的进程内。当一个进程因为“自然原因”结束（比如一个未处理的异常），它会发送一个`exit`信号。一个进程可以因为明确地发送一个`exit`信号而结束：

```elixir
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```

上面的例子中，被连接的进程由于发送了一个带有值1的`exit`信号而结束。Elixir shell会自动处理那些消息并打印在终端上。

`exit`也可以使用`try/catch`来“捕获”：

```elixir
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

不过使用`try/catch`是非常少见的，使用它捕获`exit`信号就更加少见了。

`exit`信号是Erlang虚拟机提供的容错系统的重要一部分。进程一般运行在监控树中，监控树进程会监听那些被监控进程的`exit`信号。一旦接收到`exit`信号，监控策略会重启被监控进程。

在Elixir中，这种监控系统让`try/catch`和`try/rescue`这种结构变成不常用。我们宁可让进程“立刻失败”，然后监控系统会保证在发生错误之后让我们的应用程序回到一个已知的初始状态上，也不去纠正一个错误。

## After

有时，在某些可能引发错误的操作之后，确保资源被清理干净是必要的。`try/after`结构允许你这么做。举个例子，我们可以打开一个文件，然后用一个`after`子句来关闭这个文件，即使打开文件出错也会关闭它：

```elixir
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

不管try块中的代码成功与否，`after`子句都会执行。但是注意，如果有一个连接进程存在，这个进程将退出且`after`子句不会被执行。因此`after`子句在执行上只提供了一个软性保证。幸运的是，Elixir中的文件都被连接到当前进程中，因此在当前进程崩溃的时候文件都能被正确关闭，这是独立于`after`子句存在的机制。你将在其他资源比如ETS表、套接字、端口等资源上看到类似的机制。

有时你可能想将整个函数体包装在try结构内，然后保证在try执行之后一定会执行某些代码。在这种情况下，Elixir允许你省略try这行：

```elixir
iex> defmodule RunAfter do
...>   def without_even_trying do
...>     raise "oops"
...>   after
...>     IO.puts "cleaning up!"
...>   end
...> end
iex> RunAfter.without_even_trying
cleaning up!
** (RuntimeError) oops
```

Elixir会自动包装函数体到`try`中，不管后面是`after`、`rescue`还是`catch`。

## Else

当出现`else`块时，它会在`try`块正常运行时匹配它的结果。

```elixir
iex> x = 2
2
iex> try do
...>   1 / x
...> rescue
...>   ArithmeticError ->
...>     :infinity
...> else
...>   y when y < 1 and y > -1 ->
...>     :small
...>   _ ->
...>     :large
...> end
:small
```

`else`块中的异常不会被捕获。如果`else`块没有匹配到任何模式，将抛出一个异常；这个异常将不会被当前的`try/catch/rescue/after`块所捕获。

## 变量作用域

需要牢记在心的一个很重要的事情是，定义在`try/catch/rescue/after`块中的变量不会泄露到外部环境中去。这是因为`try`块可能会失败，这会导致后面的变量永远不能被绑定到值。换句话说，下面代码是无效的：

```elixir
iex> try do
...>   raise "fail"
...>   what_happened = :did_not_raise
...> rescue
...>   _ -> what_happened = :rescued
...> end
iex> what_happened
** (RuntimeError) undefined function: what_happened/0
```

相反，你可以把`try`表达式存储到一个变量中：

```elixir
iex> what_happened =
...>   try do
...>     raise "fail"
...>     :did_not_raise
...>   rescue
...>     _ -> :rescued
...>   end
iex> what_happened
:rescued
```

到这里就结束了对`try`、`catch`和`rescue`的介绍了。你会发现相比于其他语言，它们在Elixir中的使用频率不高，虽然它们可能在一些不规范的库中被使用。
