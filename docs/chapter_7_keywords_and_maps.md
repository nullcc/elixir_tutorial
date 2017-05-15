# 关键字和映射

1.关键字列表
2.映射
3.嵌套数据结构

到目前未知，我们还没讨论过任何关联数据类型，即那些可以用一个键来获取具体值（或多个值）的数据结构。不同的编程语言对这种数据结构的称呼不同，比如字典、哈希、关联数组等。

在Elixir中，有两种主要的关联数据结构：关键字列表和映射。我们来看看它们！

## 关键字列表

在很多函数式编程语言中，经常用2元元组来表示一个键-值数据结构。在Elixir中，我们使用一个每个元组的第一个元素为原子的元组的列表来表示这种结构，叫做关键字列表：

```elixir
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
```

如在上面看到的，Elixir支持一种特殊的语法来定义这种列表：`[key: value]`。 由于关键字列表也是列表，所以对列表的所有操作对它也是可行的。比如，可以使用`++`来向关键字列表中添加新元素：

```elixir
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```

注意，在列表头部添加的元素在查找时会被优先获取到：

```elixir
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

关键字列表很重要，因为它有三个特性：

* 键必须是原子。
* 键是有序的，有开发者指定。
* 可以有多个相同的键。

比如，[Ecto库](https://github.com/elixir-lang/ecto)使用这些特性来实现一个优雅的领域专用语言来写数据库查询：

```elixir
query = from w in Weather,
      where: w.prcp > 0,
      where: w.temp < 20,
     select: w
```

这些特性让关键字列表成为Elixir中实现函数可选参数的机制。在第五章，我们谈到了`if/2`这个宏，它支持下面的语法：

```elixir
iex> if false, do: :this, else: :that
:that
```

`do`和`else`对就是关键字列表！实际上，上面的用法等同于：

```elixir
iex> if(false, [do: :this, else: :that])
:that
```

这还等同于：

```elixir
iex> if(false, [{:do, :this}, {:else, :that}])
:that
```

一般来说，当关键字列表是函数的最后一个参数时，中括号可以省略。

虽然可以对关键字列表上进行模式匹配，但实际中很少这么做，因为在列表上进行模式匹配需要左右两边列表的元素数量和顺序都要匹配：

```elixir
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```

为了操纵关键字列表，Elixir提供了[Keyword模块](https://hexdocs.pm/elixir/Keyword.html)。请记住，虽然关键字列表是简单的列表，但它的线性性能特性和普通列表一样。列表越长，查找一个键、计算列表元素个数这些操作的耗时就越长。出于这些原因，在Elixir中关键字列表一般被用来传递可选值。如果你需要存储大量元素或需要保证一个键只能对应一个值，那么你需要使用映射。

## 映射

在Elixir中，如果你需要一个键值存储，映射可以帮上忙。可以使用`%{}`语法来创建一个映射：

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
iex> map[:c]
nil
```

和关键字列表相比，我们可以看到映射的两个不同点：

* 映射允许任何值作为键。
* 映射的键是无序的。

和关键字列表不同，映射在模式匹配上用处很大。当把一个映射作为一个模式时，它总是能匹配它的一个子集：

```elixir
iex> %{} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```
如上所述，给定一个映射A，只要另一个映射B的所有键都能存在于映射A中，则二者匹配。因此，一个空映射可以匹配任何映射。

变量可用于访问、匹配和添加映射键：

```elixir
iex> n = 1
1
iex> map = %{n => :one}
%{1 => :one}
iex> map[n]
:one
iex> %{^n => :one} = %{1 => :one, 2 => :two, 3 => :three}
%{1 => :one, 2 => :two, 3 => :three}
```

[Map模块](https://hexdocs.pm/elixir/Map.html)提供了和`Keyword`模块非常相似的API来操纵映射：

```elixir
iex> Map.get(%{:a => 1, 2 => :b}, :a)
1
iex> Map.put(%{:a => 1, 2 => :b}, :c, 3)
%{2 => :b, :a => 1, :c => 3}
iex> Map.to_list(%{:a => 1, 2 => :b})
[{2, :b}, {:a, 1}]
```

映射可以使用下面的几种语法来更新一个键的值：

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}

iex> %{map | 2 => "two"}
%{2 => "two", :a => 1}
iex> %{map | :c => 3}
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

上面的语法成立的前提是给定的键存在。这种语法不能被用来添加新的键-值对。比如，上面的语法中操作`:c`会失败，因为`:c`不在映射中。

当映射中所有的键都是原子时，可以使用更简便的语法：

```elixir
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

映射的另一个有趣的性质是，它提供了`.`语法来访问原子键：

```elixir
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}

iex> map.a
1
iex> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

相较于使用Map模块中的函数，Elixir开发者一般倾向于使用`map.field`语法和模式匹配，因为这更符合Elixir的风格。[这篇博客文章](http://blog.plataformatec.com.br/2014/09/writing-assertive-code-with-elixir/)对如何在Elixir中写出更简洁和快速的代码提供了洞见和实例。

> 注意：映射是在最近才被引入Erlang虚拟机的，并在只有在Elixir v1.2之后的版本它才能高效地操作数以百万计的键。因此，如果你正在使用的是早期版本的Elixir(v1.0或v1.1)，当你需要操作至少几百个键时，你可能需要考虑使用[HashDict模块](https://hexdocs.pm/elixir/HashDict.html)

## 嵌套数据结构

我们经常在映射里嵌套映射、在映射里包含关键字列表，或者其他嵌套情况。Elixir提供了类似`put_in/2`、`update_in/2`和其他的宏来操纵嵌套数据结构，其他语言中也有类似的东西。

假设你有如下结构：

```elixir
iex> users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
[john: %{age: 27, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}]
```

这里是一个用户的关键字列表，每个元素是一个映射，其中包含名字、年龄和一个用户喜欢的编程语言列表。如果我们想访问john的年龄，我们可以写：

```elixir
iex> users[:john].age
27
```

我们可以用同样的语法来更新它的值：

```elixir
iex> users = put_in users[:john].age, 31
[john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#", "Clojure"], name: "Mary"}]
```

`update_in/2`宏也是类似的，不过它允许我们传递一个函数来控制值如何变化。比如，让我们从Mary的编程语言列表中移除“Clojure”：

```elixir
iex> users = update_in users[:mary].languages, fn languages -> List.delete(languages, "Clojure") end
[john: %{age: 31, languages: ["Erlang", "Ruby", "Elixir"], name: "John"},
 mary: %{age: 29, languages: ["Elixir", "F#"], name: "Mary"}]
```

关于`put_in/2`和`update_in/2`函数还有一些其他信息，包括允许我们一次性添加和更新数据结构的`get_and_update_in/2`函数。还有`put_in/3`、`update_in/3`和`get_and_update_in/3`允许我们动态访问数据结构。查看[核心模块文档](https://hexdocs.pm/elixir/Kernel.html)可以获得更多信息。

本章总结了Elixir中的关联数据结构。你学到了关键字列表和映射，之后你将遇到很多需要使用关联数据结构解决的问题。
