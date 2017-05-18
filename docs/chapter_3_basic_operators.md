# 第三章 基本操作符

在上一章中，我们看到Elixir提供了`+`、`-`、`*`、`/` 作为算数运算符，还有`div/2`和`rem/2`作为整型除法和取模运算函数。

Elixir还提供了`++`和`--`来操作列表：

```elixir
iex> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex> [1, 2, 3] -- [2]
[1, 3]
```

字符串连接操作要使用`<>`：

```elixir
iex> "foo" <> "bar"
"foobar"
```

Elixir还提供三个布尔运算符：`or`、'and'和`not`。这些运算符的使用必须严格遵循第一个参数必须是一个布尔值这个规则：

```elixir
iex> true and true
true
iex> false or is_atom(:example)
true
```

如果第一个参数不是布尔值将抛出异常：

```elixir
iex> 1 and true
** (BadBooleanError) expected a boolean on left-side of "and", got: 1
```

`or`和`and`是短路操作符。如果它们执行一侧的表达式之后就能得出结果则会忽略剩下的表达式：

```elixir
iex> false and raise("This error will never be raised")
false
iex> true or raise("This error will never be raised")
true
```

> 注意：如果你是一个Erlang开发者，Elixir中`and`和`or`等同于Erlang中的`andalso`和`orelse`操作符。除了这些布尔操作符，Elixir还提供`||`、`&&`和`!`，这几个操作符可以接受任何类型的值作为参数，它们会把除了`false`和`nil`以外的任何值当作`true`：

```elixir
# or
iex> 1 || true
1
iex> false || 11
11
```

```elixir
# and
iex> nil && 13
nil
iex> true && 17
17
```

```elixir
# !
iex> !true
false
iex> !1
false
iex> !nil
true
```

根据经验，当你的参数是布尔值的时候，请使用`or`、`and`和`not`。如果其中有任何一个参数是非布尔值，请使用`&&`、`||`和`!`。

Elixir还提供了`==`、 `!=`、`===`、`!==`、`<=`、`>=`、`<`、和`>`作为比较操作符：

```elixir
iex> 1 == 1
true
iex> 1 != 2
true
iex> 1 < 2
true
```

`==`和`===`的区别在于在比较整数和浮点数时的严格程度不同：

```elixir
iex> 1 == 1.0
true
iex> 1 === 1.0
false
```

Elixir中，我们可以比较两种不同数据类型的值的大小：

```elixir
iex> 1 < :atom
true
```

之所以能比较两个不同数据类型的值是因为实用主义。排序算法不需要考虑数据类型的问题。各个数据类型的大小比较方式如下：

```elixir
number < atom < reference < function < port < pid < tuple < map < list < bitstring
```

你不必去记住这个顺序，只需要知道有这个东西存在就行。

如果需要参考有关操作符（和数据排序）的信息，可以查阅(操作符文档)[http://elixir-lang.org/docs/master/elixir/operators.html]

在下一章，我们要讨论一些基本的函数、数据类型转换和一些流程控制的知识。
