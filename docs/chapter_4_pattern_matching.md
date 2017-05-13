# 模式匹配

1.模式匹配操作符
2.模式匹配
3.^操作符

本章我们将学习到在Elixir中`=`操作符实际上是一个匹配操作符，你将了解到如何使用它来进行数据结构的模式匹配。最后，我们将学习使用`^`操作符来访问先前绑定的值。

## 匹配操作符

我们已经在Elixir中使用`=`操作符来赋值很多次了：

```elixir
iex> x = 1
1
iex> x
1
```

Elixir的`=`操作符实际上被叫做匹配操作符，我们来看看为什么这么叫：

```elixir
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

注意`1 = x`是一个合法的表达式，由于表达式的左右都等于1，所以匹配成功。当两端不匹配时，会抛出MatchError异常。

一个变量只能放在`=`的左边被赋值：

```elixir
iex> 1 = unknown
** (CompileError) iex:1: undefined function unknown/0
```

由于变量`unknown`没有被赋值过，Elixir会认为你尝试调用一个叫做`unknown/0`的函数，但这个函数并不存在。

## 模式匹配

匹配操作符不仅可以用于匹配简单类型的值，还可以用来对复杂数据类型进行解构。比如，我们可以在元组上应用匹配操作符：

```elixir
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

在模式匹配时，如果两侧的值不匹配，会抛出异常，比如在匹配两个不同大小的元组时：

```elixir
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

或者当匹配两个不同类型的值时：

```elixir
iex> {a, b, c} = [:hello, "world", 42]
** (MatchError) no match of right hand side value: [:hello, "world", 42]
```

更有趣的是，我们可以匹配特定的具体值。下面的例子展示了两个元组的匹配情况，只有右边的元组的第一个值是原子`:ok`时，才能匹配成功：

```elixir
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13
```

```elixir
iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

我们来看看对列表进行模式匹配：

```elixir
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

列表还支持对列表头和列表尾进行模式匹配：

```elixir
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```
和`hd/1`和`tl/1`两个函数类似，不能对一个空列表进行列表头和列表尾的模式匹配：

```elixir
iex> [h | t] = []
** (MatchError) no match of right hand side value: []
```

`[head | tail]`这种格式不仅可以用于模式匹配，还可以在一个列表的头部添加元素：

```elixir
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0 | list]
[0, 1, 2, 3]
```

模式匹配让开发者可以很简单地解构像元组和列表这样的数据类型。就像在接下来的章节中会看到的，模式匹配是Elixir中递归的基础之一，它也适用于其他类型，比如字典和二进制。

## ^操作符

Elixir中的变量可以被重新绑定：

```elixir
iex> x = 1
1
iex> x = 2
2
```

当你想要对一个已经存在的变量进行模式匹配而不是重新绑定值时，你需要使用`^`操作符。

```elixir
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {y, ^x} = {2, 1}
{2, 1}
iex> y
2
iex> {y, ^x} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
```

由于我们已经对变量x赋值了1，最后一个例子也可以被写成：

```elixir
iex> {y, 1} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
```

如果一个变量在一个模式匹配中被引用了不止一次，则所有的引用会被绑定到相同的模式：

```elixir
iex> {x, x} = {1, 1}
{1, 1}
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

在某些情况下，你不关心某个具体值的模式。一种常见的做法是使用下划线`_`来代替它们的位置。比如，如果只对列表头感兴趣，我们可以把列表尾置为下划线：

```elixir
iex> [h | _] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

`_`是一个特殊的变量，它是不可读的。尝试读取它会造成一个未绑定变量的错误：

```elixir
iex> _
** (CompileError) iex:1: unbound variable _
```

虽然模式匹配让我们能够构建更强大的解构，但它的使用也是有限制的。比如你不能在模式匹配的左侧使用函数调用。下面的例子是错误的：

```elixir
iex> length([1, [2], 3]) = 3
** (CompileError) iex:1: illegal pattern
```

关于模式匹配的介绍到这里就结束了。我们在下一章将会看到，在很多语言结构中，模式匹配非常常见。
