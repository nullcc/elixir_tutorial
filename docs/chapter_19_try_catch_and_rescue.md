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

All Elixir code runs inside processes that communicate with each other. When a process dies of “natural causes” (e.g., unhandled exceptions), it sends an exit signal. A process can also die by explicitly sending an exit signal:

iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
In the example above, the linked process died by sending an exit signal with value of 1. The Elixir shell automatically handles those messages and prints them to the terminal.

exit can also be “caught” using try/catch:

iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
Using try/catch is already uncommon and using it to catch exits is even more rare.

exit signals are an important part of the fault tolerant system provided by the Erlang VM. Processes usually run under supervision trees which are themselves processes that listen to exit signals from the supervised processes. Once an exit signal is received, the supervision strategy kicks in and the supervised process is restarted.

It is exactly this supervision system that makes constructs like try/catch and try/rescue so uncommon in Elixir. Instead of rescuing an error, we’d rather “fail fast” since the supervision tree will guarantee our application will go back to a known initial state after the error.

After

Sometimes it’s necessary to ensure that a resource is cleaned up after some action that could potentially raise an error. The try/after construct allows you to do that. For example, we can open a file and use an after clause to close it–even if something goes wrong:

iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
The after clause will be executed regardless of whether or not the tried block succeeds. Note, however, that if a linked process exits, this process will exit and the after clause will not get run. Thus after provides only a soft guarantee. Luckily, files in Elixir are also linked to the current processes and therefore they will always get closed if the current process crashes, independent of the after clause. You will find the same to be true for other resources like ETS tables, sockets, ports and more.

Sometimes you may want to wrap the entire body of a function in a try construct, often to guarantee some code will be executed afterwards. In such cases, Elixir allows you to omit the try line:

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
Elixir will automatically wrap the function body in a try whenever one of after, rescue or catch is specified.

Else

If an else block is present, it will match on the results of the try block whenever the try block finishes without a throw or an error.

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
Exceptions in the else block are not caught. If no pattern inside the else block matches, an exception will be raised; this exception is not caught by the current try/catch/rescue/after block.

Variables scope

It is important to bear in mind that variables defined inside try/catch/rescue/after blocks do not leak to the outer context. This is because the try block may fail and as such the variables may never be bound in the first place. In other words, this code is invalid:

iex> try do
...>   raise "fail"
...>   what_happened = :did_not_raise
...> rescue
...>   _ -> what_happened = :rescued
...> end
iex> what_happened
** (RuntimeError) undefined function: what_happened/0
Instead, you can store the value of the try expression:

iex> what_happened =
...>   try do
...>     raise "fail"
...>     :did_not_raise
...>   rescue
...>     _ -> :rescued
...>   end
iex> what_happened
:rescued
This finishes our introduction to try, catch, and rescue. You will find they are used less frequently in Elixir than in other languages, although they may be handy in some situations where a library or some particular code is not playing “by the rules”.
