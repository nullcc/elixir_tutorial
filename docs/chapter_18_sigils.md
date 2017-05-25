# 第十八章 Sigils

1.正则表达式
2.字符串、字符列表和单词列表sigils
  2.1 字符串
  2.2 字符列表
  2.3 单词列表sigils
3.插值和转义sigils
4.自定义sigils

我们已经知道在Elixir中，双引号包裹的是字符串，单引号包裹的是字符列表。然而，这只涵盖了Elixir文字表示的数据结构的一部分。更经常使用的是用`:atom`创建的原子。

Elixir的一个目标之一是可扩展性：开发者应该能够针对任何特定的领域扩展语言。计算机科学是个范围很宽泛的领域，一个语言的核心不可能处理所有领域的问题，相反，我们的最佳方案是让语言具有可扩展性，让开发者、公司和社区能够将语言扩展到相关领域。

在本章中，我们将探索sigils，这是语言提供的文本表示的机制之一。sigils以波浪号(`~`)开头，后面跟一个字符串（用来标识sigils），然后是一个分隔符；可选地，可以在最后的分隔符后加上修饰符。

## 正则表达式

Elixir中最常见的sigil是`~r`，被用来创建[正则表达式](https://en.wikipedia.org/wiki/Regular_Expressions)

```elixir
# A regular expression that matches strings which contain "foo" or "bar":
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

Elixir提供了兼容Perl的正则表达式，这由PCRE库实现。举个例子，`i`修饰符使正则表达式大小写不敏感：

```elixir
iex> "HELLO" =~ ~r/hello/
false
iex> "HELLO" =~ ~r/hello/i
true
```

查看[`Regex`模块](https://hexdocs.pm/elixir/Regex.html)可以获得更多正则表达式修饰符和所支持操作的信息。

到目前为止，所有例子都使用`/`作为正则表达式的分隔符。然而sigils支持8中不同的分隔符：

```elixir
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

支持多种不同分隔符的理由是为了给在需要转义分隔符的时候提供其他可选方式。比如，一个有斜杠的正则表达式`~r(^https?://)`就比写成`~r/^https?:\/\//`可读性更好。类似地，如果一个正则表达式有斜杠且还有分组（即使用了`()`），你就可以选择双引号而不是圆括号。

## 字符串、字符列表和单词列表sigils

除了正则表达式，Elixir还有三种其他的sigils。

### 字符串

`~s`sigil用来生成字符串，和双引号类似。当一个字符串包含双引号时，`~s`sigil非常有用：

```elixir
iex> ~s(this is a string with "double" quotes, not 'single' ones)
"this is a string with \"double\" quotes, not 'single' ones"
```

## 字符列表

`~c`sigil在生成包含单引号的字符列表时非常有用：

```elixir
iex> ~c(this is a char list containing 'single quotes')
'this is a char list containing \'single quotes\''
```

## 单词列表

`~w`sigil用来生成单词的列表（单词就是普通的字符串）。在`~w`sigil中，单词之间被空格分隔。

```elixir
iex> ~w(foo bar bat)
["foo", "bar", "bat"]
```

`~w`sigil还接受`c`、`s`和`a`修饰符（分别对应字符列表、字符串和原子），修饰符指定了结果列表中元素的数据类型：

```elixir
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

## 插值和转义sigils

除了小写的sigils，Elixir还有大写的sigils来支持转义字符和插值。虽然`~s`和`~S`都会返回字符串，但是前者允许有转义码和插值，后者则不行：

```elixir
iex> ~s(String with escape codes \x26 #{"inter" <> "polation"})
"String with escape codes & interpolation"
iex> ~S(String without escape codes \x26 without #{interpolation})
"String without escape codes \\x26 without \#{interpolation}"
```

下列的转义码能被用于字符串和字符列表：

* `\\` – 单个反斜杠
* `\a` – 响铃符
* `\b` – 退格符
* `\d` - 删除符
* `\e` - 转义符
* `\f` - 换页符
* `\n` – 换行符
* `\r` – 回车符
* `\s` – 空白符
* `\t`– 制表符
* `\v` – 垂直制表符
* `\0` - 空字符
* `\xDD` - 以十六进制表示一个字节（例如`\x13`）
* `\uDDDD` and `\u{D...}` - 以十六进制表示一个Unicode码点（例如`\u{1F600}`）

除了上面说的，双引号包裹的字符串内的双引号需要被转义成`\"`，类似地，单引号包裹的字符列表内的单引号需要被转义成`\'`。然而，比较好的方式是改变分隔符来避免做这种转义。

sigils还支持heredocs，使用三个双引号或三个单引号作为分隔符：

```elixir
iex> ~s"""
...> this is
...> a heredoc string
...> """
```

heredoc sigils最常用的地方是写文档的时候。比如，在文档中写需要转义的字符很容易出错，因为你需要双重转义一些字符：

```elixir
@doc """
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\\\"foo\\\"")
    "'foo'"

"""
def convert(...)
```

通过使用`~S`，可以完全避免这种问题：

```elixir
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

## 自定义sigils

正如本章开头所暗示的，Elixir中的sigils是可扩展的。事实上，使用sigil`~r/foo/i`相当于使用一个二进制数据和一个字符列表作为参数来调用`sigil_r`：

```elixir
iex> sigil_r(<<"foo">>, 'i')
~r"foo"i
```

可以使用`sigil_r`来访问`~r`sigil的文档：

```elixir
iex> h sigil_r
...
```

我们还可以通过实现函数，遵循`sigil_{identifier}`模式来定义自己的sigils。举个例子，我们来实现`~i`sigil，它返回一个整型数（使用`n`修饰符会让它变成负数）：

```elixir
iex> defmodule MySigils do
...>   def sigil_i(string, []), do: String.to_integer(string)
...>   def sigil_i(string, [?n]), do: -String.to_integer(string)
...> end
iex> import MySigils
iex> ~i(13)
13
iex> ~i(42)n
-42
```

sigils也可以在宏的帮助下做一些编译期的工作。举个例子，Elixir的正则表达式在编译期间从源码被编译成一个有效的表示形式，因此在运行时需要跳过这个步骤。如果你对这个主题有兴趣，我们推荐你多看看和宏有关的内容并深入了解一下`Kernel`模块中如何实现sigils（`sigil_*`在哪里被定义的）。
