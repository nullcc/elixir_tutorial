# 递归

1.使用递归实现循环
2.归约和映射算法

## 使用递归实现循环

由于不可变性，Elixir（和任何函数式编程语言一样）中的循环和其他编程语言有所不同。比如，C语言中的循环可以这么写：

```c
for(i = 0; i < sizeof(array); i++) {
  array[i] = array[i] * 2;
}
```

上面这个例子中，我们改变了数组和循环变量`i`。但在Elixir中我们不能改变变量的值。函数式编程语言依赖递归：一个函数递归调用自身，直到达到结束条件为止。在这个过程中不能改变数据。下面的例子可以对给定的字符串打印任意的次数：

```elixir
defmodule Recursion do
  def print_multiple_times(msg, n) when n <= 1 do
    IO.puts msg
  end

  def print_multiple_times(msg, n) do
    IO.puts msg
    print_multiple_times(msg, n - 1)
  end
end

Recursion.print_multiple_times("Hello!", 3)
# Hello!
# Hello!
# Hello!
```

和`case`类似，一个函数可以有多个子句。如果一个子句的模式能被传递给这个函数的参数所匹配且求值结果为`true`，则这个子句将被执行。

上面的例子中，当函数`print_multiple_times/2`被调用时，n等于3。

第一个子句拥有一个哨兵，表示只有当`n`小于或等于1的时候才执行这个子句的代码。当前这个子句并不匹配，因此Elixir继续往下处理下个子句。

第二个子句匹配成功且没有任何哨兵，因此第二个子句将被执行。它打印我们传递的`msg`参数，并且以参数`n - 1(2)`作为第二个参数调用自身。

再接下来函数`print_multiple_times/2`再次被调用，此时第二个参数的值为1。由于现在`n`的值为1，函数`print_multiple_times/2`的第一个子句中的哨兵求值为`true`，所以第一个子句被执行，打印`msg`然后结束整个过程。

我们像这样定义`print_multiple_times/2`函数，就能在不管第二个参数的值是什么的情况下，在一次执行中都能触发子句一和子句二两个的其中一个，这能确切地保证函数每被执行一次都更加接近它的结束条件。

## 归约和映射算法

我们来看看如何利用递归的能力来求和一个数字列表：

```elixir
defmodule Math do
  def sum_list([head | tail], accumulator) do
    sum_list(tail, head + accumulator)
  end

  def sum_list([], accumulator) do
    accumulator
  end
end

IO.puts Math.sum_list([1, 2, 3], 0) #=> 6
```

我们用列表`[1, 2, 3]`和初始值`0`来调用`sum_list`函数。调用时将会对逐一对所以子句进行匹配直到找到能够匹配的子句。在这个例子中，列表`[1, 2, 3]`匹配到模式`[head | tail]`，`head`被绑定到`1`，`tail`被绑定到`[2, 3]`；`accumulator`为0

之后，执行`head + accumulator`且再次递归调用`sum_list`，把`tail`作为第一个参数传入。`tail`将再次匹配模式`[head | tail]`直到为空列表：

```elixir
sum_list [1, 2, 3], 0
sum_list [2, 3], 1
sum_list [3], 3
sum_list [], 6
```

当列表变为空时，它将匹配最后一条子句，返回最终结果6。

这种把一个列表逐步归约到一个值的做法叫归约算法，这是函数式编程语言的核心。

如果我们想加倍列表中的所有值呢？

```elixir
defmodule Math do
  def double_each([head | tail]) do
    [head * 2 | double_each(tail)]
  end

  def double_each([]) do
    []
  end
end
iex math.exs
iex> Math.double_each([1, 2, 3]) #=> [2, 4, 6]
```

这里我们用递归来遍历一个列表，加倍每个元素并返回一个新列表。这种把一个列表映射到定一个列表的做法叫映射算法。

递归和尾调用优化是Elixir的重要部分，经常被用于创建循环。然而，在Elixir编程中你将很少使用递归来处理列表。

我们将在下一章介绍`Enum`模块，提供了很多操作列表的方便方法。上面的例子可以改写成这样：

```elixir
iex> Enum.reduce([1, 2, 3], 0, fn(x, acc) -> x + acc end)
6
iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
[2, 4, 6]
```

或者使用函数捕获语法：

```elixir
iex> Enum.reduce([1, 2, 3], 0, &+/2)
6
iex> Enum.map([1, 2, 3], &(&1 * 2))
[2, 4, 6]
```

让我们更进一步地来看看`Enumerable`，当我们讨论它时，还要看来看看它的惰性版本：`Stream`。
