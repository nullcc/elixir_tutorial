# 第十一章 进程

1.`spawn`函数
2.`send`函数和`receive`函数
3.连接进程
4.任务
5.进程状态

在Elixir中，所有代码都运行在进程中。进程之间是相互隔离的，它们并发地运行且通过消息传递彼此沟通。进程不仅仅是Elixir并发的基础，它们还有助于构建分布式和容错程序。

不要混淆Elixir的进程和操作系统中的基础。在使用内存和CPU方面，Elixir中的进程是极其轻量级的（这不像其他语言中的线程那样开销比较大）。正由于这个原因，在Elixir里运行成百上千个进程非常正常。

在本章中，我们将学习生成新进程的基本方法，以及在进程间发送和接收消息。

## `spawn`函数

生成新进程的基本方式是使用默认导入的`spawn/1`函数：

```elixir
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

`spawn/1`函数接受一个函数作为参数，它将在一个新进程中运行这个函数。

注意`spawn/1`函数返回一个PID（进程标识符）。从这点上来说，你创建的这个进程很可能已经执行结束了。这个被新创建出来的进程将执行给定的函数，然后在函数执行完毕后退出：

```elixir
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

>注意：你可能会获得和本教程例子中不同的进程标识符。我们可以使用`self/0`获取当前进程的PID：

```elixir
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

当我们在进程间发送和接收消息时，进程会变得很有趣。

## `send`函数和`receive`函数

我们可以使用`send/2 `函数和`receive/1`函数在进程间发送和接收消息：

```elixir
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

当给一个进程发送消息时，消息将被存储在进程的邮箱中。`receive/1`函数会阻塞进程直到当前进程在邮箱中匹配到一条消息。和`case/2`类似，`receive/1`支持哨兵和多重子句。

发送消息的进程不会因为调用`send/2`而阻塞，它把消息放到接收进程的邮箱中然后继续往下执行。特别地，一个进程可以对自己发送消息。

当邮箱中没有任何匹配的消息时，当前进程会等待直到一个匹配的消息到达。我们也可以指定一个等待超时时间：

```elixir
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

如果期望匹配的消息已经在邮箱中时，可以指定一个等待时间为0的超时时间。

让我们试试在进程间发送消息：

```elixir
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

`inspect`函数会把数据结构以字符串的形式显示出来，一般作为打印使用。需要注意的时，当接收进程中的块被执行时，发送进程很可能已经退出，因为发送一个消息是发送进程的唯一指令。

在shell中，辅助函数`flush/0`非常有用。它刷新和打印所有邮箱中的信息。

```elixir
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## 连接进程

在Elixir中，大部分情况下我们会创建连接进程。在我们展示`spawn_link/1`函数的例子之前，让我们来看看当调用`spawn/1`失败时会发生什么：

```elixir
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Process #PID<0.58.00> raised an exception
** (RuntimeError) oops
    :erlang.apply/2
```

生成进程失败只会记录一个错误，但父进程还在继续运行。这是因为进程之间是相互隔离的。如果我们想要让一个进程中的失败影响到另一个进程，我们需要连接它们。这需要用到`spawn_link/1`函数：

```elixir
iex> spawn_link fn -> raise "oops" end
#PID<0.41.0>

** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

由于进程之间被连接，我们现在会看到父进程shell的消息，它从其他进程接收到一个退出信号从而被终止。`IEx`会检测到这种情况并开始一个新的shell会话。

也可以手动调用`Process.link/1`来连接进程。我们建议你熟悉下`Process`模块中提供的其他功能。

在构建容错系统时进程和连接起了很重要的作用。Elixir的进程时相互隔离的且它们之间默认不会共享任何东西。因此。一个进程中的错误永远不会导致另一个进程被破坏或崩溃。然而，连接允许进程之间在出错时相互影响。我们经常把进程和它的监控进程连接在一起，这样监控进程就可以检测到那个进程的退出或者在那个进程中生成新进程这些信息。

在其他语言中，我们需要捕获/处理异常，在Elixir中我们可以让进程直接失败，因为它的监控进程会正确地重启我们的系统。“任其失败”是在编写Elixir软件时的常用理念！

`spawn/1`和`spawn_link/1`是Elixir中创建进程的基本原语。虽然迄今为止我们只用了它们来创建进程，但是大多数时候我们都会使用构建在它们之上的更抽象的方法来创建进程。让我们来看看其中的一个，叫做任务。

## 任务

任务构建在spawn函数簇之上，它提供更好的错误报告和内省信息：

```elixir
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
** (RuntimeError) oops
    (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
    (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []
```

我们使用返回值为`{:ok, pid}`而不是PID的`Task.start/1`和`Task.start_link/1`来替代`spawn/1`和`spawn_link/1`。这就是在监控树中使用任务的原因。此外，任务提供了更方便的函数，像`Task.async/1`和`Task.await/1`等函数来简化分布式程序的开发。

我们将在《Mix和OTP指南》中学习这些函数，现在只需要直到使用任务来处理进程可以获得更友好的出错报告。

## 进程状态

到目前为止，我们还没有讨论进程的状态。如果我们构建的应用是状态依赖的，比如，你需要应用程序配置，或你需要解析一个文件然后把它存储在内存中，我们应该怎么做呢？

这种情况下一般使用进程处理。我们可以使用一个无限循环的进程来维护状态、发送消息和接收消息。一个例子是，我们写一个模块，它生成新进程作为一个键-值存储引擎，其中键-值信息保存在`kv.exs`文件中：

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end
```

注意，`start_link`函数会生成一个新进程，进程中运行`loop/1`函数，初始参数为一个空映射。`loop/1`函数会等待消息的到来并根据消息来执行相应的操作。当收到`:get`消息时，它会向调用者发送一个消息并调用`loop/1`函数，继续等待一个新消息。当收到`:put`消息时，它将把新的键值对添加到映射中，并以这个映射调用`loop/1`函数。

试着运行`iex kv.exs`：

```elixir
iex> {:ok, pid} = KV.start_link
{:ok, #PID<0.62.0>}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
nil
:ok
```

依赖是，进程中的映射里没有任何键，所以发送一个`:get`消息给进程并刷新当前进程的邮箱会返回`nil`。尝试一下发送`:put`消息给进程然后再试一次：

```elixir
iex> send pid, {:put, :hello, :world}
{:put, :hello, :world}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
:world
:ok
```

请留意进程是如何保持状态并且在我们发送消息的时候更新和获取状态的。实际上，任何知道上面的PID的进程都能够操纵这个状态。

还可以给PID注册一个名字，只要知道这个名字就能给该进程发送消息：

```elixir
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
:world
:ok
```

在Elixir应用程序中，使用进程来维护状态和给进程注册名字是非常常见的模式。然而，大多数时候，我们都不需要手动实现上面展示的这种模式，而是使用很多Elixir中的抽象方式。比如，Elixir提供了[代理](https://hexdocs.pm/elixir/Agent.html)这种对状态的简单抽象：

```elixir
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

可以给`Agent.start_link/2`函数提供一个`:name`选项来自动注册名字。除了代理以外，Elixir还提供了一个API来构建通用服务器（被称为`GenServer`）、任务和其他很多抽象，这些抽象底层都由进程驱动。和监控树由关的知识将在《Mix和OTP指南》中进行介绍，这个指南中介绍了如何从零开始构建一个完整的Elixir应用程序。

下一章将讨论Elixir输入/输出的知识。
