# case、cond和if

1.`case`
2.哨兵子句中的的表达式
3.`cond`
4.`if`和`unless`
5.`do/end`块

本章我们将学习`case`、`cond`和控制流结构


## `case`

`case`允许我们使用多个模式去匹配一个值，直到找到可以匹配的那个模式：

```elixir
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
"This clause will match and bind x to 2 in this clause"
```

如果你要对一个已经存在的变量进行模式匹配，你需要使用`^`操作符：

```elixir
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Won't match"
...>   _ -> "Will match"
...> end
"Will match"
```

```elixir
哨兵子句还可以有额外的条件：

iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Would match, if guard condition were not satisfied"
...> end
"Will match"
```

第一个哨兵子句只有在x是正数的情况下才会被匹配到。

## 哨兵子句中的表达式

Elixir默认允许以下表达式出现在哨兵子句中：

* 比较操作符 (`==`, `!=`, `===`, `!==`, `>`, `>=`, `<`, `<=`)
* 布尔操作符 (`and`, `or`, `not`)
* 算数运算符 (`+`, `-`, `*`, `/`)
* 一元算数运算符 (`+`, `-`)
* 二进制级联操作符 (`<>`)
* 使用`in`操作符时右边需要是一个范围或这列表
* 以下是所有的类型检查函数：

`is_atom/1`
`is_binary/1`
`is_bitstring/1`
`is_boolean/1`
`is_float/1`
`is_function/1`
`is_function/2`
`is_integer/1`
`is_list/1`
`is_map/1`
`is_nil/1`
`is_number/1`
`is_pid/1`
`is_port/1`
`is_reference/1`
`is_tuple/1`

* 还有以下这些函数：

`abs(number)`
`binary_part(binary, start, length)`
`bit_size(bitstring)`
`byte_size(bitstring)`
`div(integer, integer)`
`elem(tuple, n)`
`hd(list)`
`length(list)`
`map_size(map)`
`node()`
`node(pid | ref | port)`
`rem(integer, integer)`
`round(number)`
`self()`
`tl(list)`
`trunc(number)`
`tuple_size(tuple)`

用户可以定义自己的哨兵。比如，`Bitwise`模块定义了一系列函数和操作符作为哨兵：`bnot`, `~~~`, `band`, `&&&`, `bor`, `|||`, `bxor`, `^^^`, `bsl`, `<<<`, `bsr`, `>>>`.

请注意虽然`and`、`or`和`not`这些布尔运算符允许出现在哨兵中，但`&&`、`||`和`!`操作符不可以。

请记住哨兵中的错误不会泄露到外部，而是导致哨兵失败：

```elixir
iex> hd(1)
** (ArgumentError) argument error
iex> case 1 do
...>   x when hd(x) -> "Won't match"
...>   x -> "Got #{x}"
...> end
"Got 1"
```

如果没有匹配到任何子句，会导致抛出异常：

```elixir
iex> case :ok do
...>   :error -> "Won't match"
...> end
** (CaseClauseError) no case clause matching: :ok
```

注意匿名函数也可以有多个哨兵和子句：

```elixir
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```

匿名函数中的子句的参数数量必须一致，否则会导致错误：

```elixir
iex> f2 = fn
...>   x, y when x > 0 -> x + y
...>   x, y, z -> x * y + z
...> end
** (CompileError) iex:1: cannot mix clauses with different arities in function definition
```

## `cond`

当你需要匹配不同的值时，`case`很有用。然而，在很多情况下，我们想要检查一些不同的条件并且找到第一个成立的条件。这种情况下，可以使用`cond`：

```elixir
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

这等同于许多命令语言中的`else if`子句（虽然在这里很少使用）。

如果所有条件都失败，会抛出异常（`CondClauseError`）。所以有必要在最后添加一个默认的条件，它总是成立的：

```elixir
iex> cond do
...>   2 + 2 == 5 ->
...>     "This is never true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   true ->
...>     "This is always true (equivalent to else)"
...> end
"This is always true (equivalent to else)"
```

最后，请注意`cond`会把除了`nil`和`false`的所有值认为是`true`：

```elixir
iex> cond do
...>   hd([1, 2, 3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

## `if`和`unless`

除了`case`和`cond`，Elixir还提供`if/2`和`unless/2`两个宏用来处理只有一个条件的情况：

```elixir
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

如果传递给`if/2`的条件返回`false`或`nil`，`do/end`块中的代码就不会被执行，并且整个结构会返回`nil`。`unless/2`则相反。

也可以使用`else`块：

```elixir
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```

> 注意：一个关于`if/2`和`unless/2`有趣的事情是，在Elixir中它们是以宏的形式被实现的；它们不像在其他很多语言中是以语言语法结构的形式被提供的。你可以查看核心模块文档来获得`if/2`的相关内容。核心模块中哥还定义了像`+/2`这类操作符和像`is_function/2`这类函数，核心模块会被默认导入，你可以直接使用它。

## `do/end`块

我们已经学习了四种控制结构：`case`、`cond`、`id`和`unless`，它们都被包裹在`do/end`块中。它还可以被写成如下几种形式：

```elixir
iex> if true, do: 1 + 2
3
```

注意到上面的例子在`true`和`do`之间有一个逗号，这是因为这里使用了Elixir的标准语法，使用逗号分隔参数。我们把这种语法称为关键字列表。我们也可以使用关键字列表来传递`else`：

```elixir
iex> if false, do: :this, else: :that
:that
```

`do/end`块是一个构建在关键字上的便捷语法形式。这就是为什么`do/end`块不需要一个逗号来分隔前一个参数和块。这种语法很有用，因为它十分简洁。下面的代码是等价的：

```elixir
iex> if true do
...>   a = 1 + 2
...>   a + 10
...> end
13
iex> if true, do: (
...>   a = 1 + 2
...>   a + 10
...> )
13
```

请记住，当使用`do/end`块时，它会被绑定到最外层的函数调用上。比如，下列表达式：

```elixir
iex> is_number if true do
...>  1 + 2
...> end
** (CompileError) undefined function: is_number/2
```

会被解析成：

```elixir
iex> is_number(if true) do
...>  1 + 2
...> end
** (CompileError) undefined function: is_number/2
```

这会导致一个未定义的函数错误，因为这个调用传入了两个参数，但并不存在`is_number/2`这个函数。`if true`这个表达式是不合法的，因为它需要一个块。但由于`is_number/2`的元数不匹配，Elixir不会产生函数调用。

可以明确地添加一个括号来把块绑定到`if`：

```elixir
iex> is_number(if true do
...>  1 + 2
...> end)
true
```

关键字列表在Elixir中很重要，它在很多函数和宏中非常常见。我们将在后面的章节中继续讨论它。现在是时候来讨论一下“二进制、字符串和字符列表”了。
