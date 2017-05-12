# 基本类型

1. 基本算数运算
2. 标记和识别函数
3. 布尔值
4. 原子
5. 字符串
6. 匿名函数
7. (链表)列表
8. 元组
9. 使用列表还是元组？

本章我们将学习更多Elixir的基本类型：整型、浮点型、布尔型、原子类型、字符串、列表和元组。

以下是一些基本类型的例子：

```elixir
iex> 1          # integer 整型
iex> 0x1F       # integer 整型
iex> 1.0        # float   浮点型
iex> true       # boolean 布尔型
iex> :atom      # atom / symbol 原子类型
iex> "elixir"   # string 字符串
iex> [1, 2, 3]  # list 列表
iex> {1, 2, 3}  # tuple 元组
```

## 基本算数运算

打开iex，输入以下表达式：

```elixir
iex> 1 + 2
3
iex> 5 * 5
25
iex> 10 / 2
5.0
```

注意`10 / 2`返回的是浮点型的5.0尔不是整型5，这是有意而为之的。在Elixir中，操作符/总是返回一个浮点型的结果。如果你想要的是整型除法或获得余数，你可以使用`div`和`rem`函数：

```elixir
iex> div(10, 2)
5
iex> div 10, 2
5
iex> rem 10, 3
1
```

注意Elixir允许你在调用一个具名函数的时候省略圆括号。这种特性能让在书写声明和流程控制结构时语法更加简洁。

Elixir也支持一些快捷符号来使用二进制、八进制和十六进制数：

```elixir
iex> 0b1010
10
iex> 0o777
511
iex> 0x1F
31
```

浮点数需要在至少一个数字后面跟上小数点，也可以使用`e`以指数形式来表示：

```elixir
iex> 1.0
1.0
iex> 1.0e-10
1.0e-10
```

Elixir的浮点数使用64位的精度表示。

你可以使用`round`函数来获得最接近一个浮点数的整型数，或者使用`trunc`函数来获得一个浮点数的整数部分。

```elixir
iex> round(3.58)
4
iex> trunc(3.58)
3
```

## 标记和识别函数

Elixir中的函数使用它的函数名和元数进行标记识别。函数的元数是指函数的入参个数。从这一点上，我们将同时使用函数名和参数个数来描述一个函数的功能。`round/1`表示这个函数的名称是`round`，它有一个入参，而`round/2`表示的是另外一个函数(不存在的)，它和`round/1`同名但有两个入参。

## 布尔值

Elixir支持`true`和`false`两种布尔值：

```elixir
iex> true
true
iex> true == false
false
```

Elixir提供了很多谓词函数来检查一个值的类型。比如，`is_boolean/1`函数可以被用来检查给定的值是否是一个布尔值：

```elixir
iex> is_boolean(true)
true
iex> is_boolean(1)
false
```

你还可以分别用`is_integer/1`、`is_float/1`或者`is_number/1`来检查一个给定值是否是一个整型、一个浮点型，或者两者其中之一。

> 注意：任何时候你都可以在iex中输入`h()`来打印信息来查看如何使用shell。`h`还可以被用来查看函数的文档。比如，输入`h is_integer/1`会打印出函数`is_integer/1`的文档。也可以用它来获得操作符和其他结构的帮助信息(试试`h ==/2`)

## 原子

Atoms are constants where their name is their own value. Some other languages call these symbols:

原子是常量，它的名称就是它的值。有些编程语言也把它们叫做符号(symbols)：

```elixir
iex> :hello
:hello
iex> :hello == :world
false
```

布尔值`true`和`false`实际上也是原子：

```elixir
iex> true == :true
true
iex> is_atom(false)
true
iex> is_boolean(:false)
true
```

## 字符串

Elixir中的字符串需要用一对双引号包裹，它们被编码成UTF-8：

```elixir
iex> "hellö"
"hellö"
```

> 注意：如果你在Windows环境下，你有机会改变终端使用的字符集，不使用UTF-8。你可以在进入IEx之前在当前命令行下运行`chcp 65001`。

Elixir还支持字符串插值：

```elixir
iex> "hellö #{:world}"
"hellö world"
```

字符串内还可以有换行符，你可以使用转义符号来使用：

```elixir
iex> "hello
...> world"
"hello\nworld"
iex> "hello\nworld"
"hello\nworld"
```

你可以使用IO模块中的`IO.puts/1`函数来打印字符串：

```elixir
iex> IO.puts "hello\nworld"
hello
world
:ok
```

注意`IO.puts/1`函数会在打印结束后返回原子`:ok`作为返回值。

Elixir中的字符串在内部被表示成字节序列：

```elixir
iex> is_binary("hellö")
true
```

我们还可以获得一个字符串的字节数：

```elixir
iex> byte_size("hellö")
6
```

注意在上面的字符串中，虽然只有5个字符，但它的字节数是6。那是因为字符“ö”在UTF-8中需要使用2个字节来表示。我们可以使用`String.length/1 `函数来基于字符数获得字符串的真实长度：

```elixir
iex> String.length("hellö")
5
```

[String模块](https://hexdocs.pm/elixir/String.html)包含了很多在Unicode标准下操作字符串的函数：

```elixir
iex> String.upcase("hellö")
"HELLÖ"
```

## 匿名函数

匿名函数可以被内联创建，它被关键字`fn`和`end`包裹。

```elixir
iex> add = fn a, b -> a + b end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> add.(1, 2)
3
iex> is_function(add)
true
iex> is_function(add, 2) # check if add is a function that expects exactly 2 arguments
true
iex> is_function(add, 1) # check if add is a function that expects exactly 1 argument
false
```

在Elixir中，函数是一等公民，这意味着函数可以像整型和字符串那样被作为参数传递给其他函数。比如，把一个函数作为参数传递给`is_function/1`函数将会返回`true`。我们还可以调用`is_function/2`函数来检查函数的元数。

需要注意在调用匿名函数时，在函数名称和括号之间需要有一个`.`。这个点保证在调用一个匿名函数`add`和调用一个具名函数`add/2`之间没有歧义。在这种情况下，Elixir明确区别了匿名函数和具名函数的使用方式。我们将在第八章探索它们不不同点。

匿名函数是一个闭包，正因如此它们可以在定义函数时访问作用域内的变量。让我们定义一个新的匿名函数，它在内部使用我们刚才定义的`add`匿名函数：

```elixir
iex> double = fn a -> add.(a, a) end
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> double.(2)
4
```

记住，一个在函数内被赋值的变量不会影响到外部环境：

```elixir
iex> x = 42
42
iex> (fn -> x = 0 end).()
0
iex> x
42
```

## (链表)列表

Elixir使用方括号来表示一个列表类型的值，列表中的值可以是任何类型：

```elixir
iex> [1, 2, true, 3]
[1, 2, true, 3]
iex> length [1, 2, 3]
3
```

两个列表的级联和相减可以使用`++/2`和`--/2`操作符：

```elixir
iex> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex> [1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

在整个教程中，我们会经常讨论到列表的头和尾。列表的头是指列表的第一个元素，列表的尾是指列表除了头以外的其余部分。它们可以使用`hd/1`和`tl/1`来获取。试试看获取一个列表的头和尾：

```elixir
iex> list = [1, 2, 3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
```

获取空列表的头或尾会抛出错误：

```elixir
iex> hd []
** (ArgumentError) argument error
```

有时候你创建一个列表，会得到一个被单引号包裹的值。比如：

```elixir
iex> [11, 12, 13]
'\v\f\r'
iex> [104, 101, 108, 108, 111]
'hello'
```

当Elixir发现一个列表中的值是可打印的ASCII码的数字时，Elixir会把它们以字符列表的形式打印出来（字符字面量的列表）。在处理Erlang代码的时候，字符列表很常见。当你在IEx中看到一个值，但不确定它具体是什么时，你可以使用`i/1`来获取关于它的信息：

```elixir
iex> i 'hello'
Term
  'hello'
Data type
  List
Description
  ...
Raw representation
  [104, 101, 108, 108, 111]
Reference modules
  List
```

请记住，在Elixir中单引号和双引号表示的意思是不同的，因为它们表示的是不同的类型：

```elixir
iex> 'hello' == "hello"
false
```

单引号表示的是字符列表，双引号表示字符串。我们将在“二进制、字符串和字符列表”一章中详细讨论它们。

## 元组

Elixir使用花括号来定义元组。和列表类似，元组可以包含任何值：

```elixir
iex> {:ok, "hello"}
{:ok, "hello"}
iex> tuple_size {:ok, "hello"}
2
```

元组在内存中存储的元素是连续的。这意味着使用索引访问一个元组的元素或获取元组大小是很快的操作。索引的起始值是0：

```elixir
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
iex> tuple_size(tuple)
2
```

可以使用`put_elem/3`函数在元组的指定索引处添加一个元素：

```elixir
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> put_elem(tuple, 1, "world")
{:ok, "world"}
iex> tuple
{:ok, "hello"}
```

请注意`put_elem/3`会返回一个新的元组。由于Elixir中的数据类型是不可变的，因此原始元组的值不会被改变。由于这种不可变性，Elixir使你不用担心你的代码在某处修改了你的数据。

## 使用列表还是元组？

列表和元组有哪些区别？

列表在内存中是用链表实现的，这说明列表中的每个元素包含一个值和一个指向下个元素的指针。我们把这种值/指针对称为`cons cell`：

```elixir
iex> list = [1 | [2 | [3 | []]]]
[1, 2, 3]
```

这意味着获取一个列表的长度是一个线性操作：我们需要按顺序遍历整个列表来计算它的大小。如果我们只是在列表的头增加元素，那么更新一个列表是很快速的操作：

```elixir
iex> [0 | list]
[0, 1, 2, 3]
```

另一方面，元组的元素在内存中是连续存储的。这意味着获取一个元组的大小或使用索引获取其中的元素是非常快速的操作。然而，更新或添加新元素对元组来说是比较昂贵的操作，因为这需要从内存中复制整个元组。

这些性能特定决定了这些数据结构的使用方式。元组的典型应用是在函数中返回额外的信息。比如，`File.read/1`函数可以用来读取文件内容，它返回一个元组：

```elixir
iex> File.read("path/to/existing/file")
{:ok, "... contents ..."}
iex> File.read("path/to/unknown/file")
{:error, :enoent}
```

如果给定的文件路径存在，`File.read/1`会返回一个元组，第一个元素是`:ok`，第二个元素是文件内容。如果文件路径不存在，会返回`:error`和错误描述。

大多数时候，Elixir都会引导你使用正确的方法做事。比如，有一个`elem/2`函数来获取元组中指定位置的元素，但对于列表来说并没有类似的函数：

```elixir
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
```

当对一个数据结构计算大小时，Elixir遵守一个简单的规则：如果函数名数叫做`size`，那么该操作的耗时是常量时间（这个值被预先计算出来了），如果叫做`length`，那么该操作的耗时是线性的（数据结构越大耗时越长）。一个记忆技巧是，`length`和`linear`都以`l`开头。

比如，到现在为止我们使用过了4个计算数据结构大小的函数：`byte_size/1`（计算字符串的字节数），`tuple_size/1`（计算元组大小），`length/1`（计算列表长度）和`String.length/1`（计算字符串的字符个数）。我们使用`byte_size`来获取字符串的字节数，这是一个高效的操作。另一方面，获取unicode字符的数目，使用`String.length`就可能比较低效，因为需要遍历整个字符串。

Elixir还提供`Port`、`Reference`和`PIDS`这些数据类型（通常被应用在进程间通讯中），我们在讨论进程的时候会讲到它们。现在，让我们来学习一些基本的操作符。
