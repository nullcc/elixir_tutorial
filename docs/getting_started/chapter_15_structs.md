# 第十五章 结构体

Default values and required keys

1.定义结构体
2.访问和更新结构体
3.结构体的底层是基于裸映射的
4.键的默认值和设置必须指定值的键

在第七章中我们学习了映射：

```elixir
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

结构体是基于映射的扩展结构，它提供了编译时检查和默认值。

## 定义结构体

使用`defstruct`来定义一个结构体：

```elixir
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

使用`defstruct`配合关键字列表来定义结构体，定义里每个字段的值是该字段的默认值。

结构体的名字就是它所在模块的名字。在上面的例子中，我么定义了一个叫做`User`的结构体。

现在我们可以使用类似创建映射的语法来创建`User`结构体：

```elixir
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

结构体提供可编译期检查来保证只有在结构体中定义过的字段才可以存在于结构体内：

```elixir
iex> %User{oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "John"}
```

## 访问和更新结构体

之前我们讨论映射时，我们已经展示了如何访问和更新一个映射中的字段。这些操作（使用相同的语法）也可以使用在结构体上：

```elixir
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "Meg"}
```

当使用更新语法（`|`）时，虚拟机知道不会有新的键被加入到结构体中，这将允许映射在底层共享内存中的结构。上面的例子中，`john`和`meg`在内存中共享相同的键结构。

结构体还能被用来进行模式匹配，这里有两个要求，一是相应键对应的值必须匹配，二是两个结构必须具有相同类型的匹配值。

```elixir
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## 结构体的底层是基于裸映射的

上面例子模式匹配成功，因为结构体的底层实现是映射，具有固定的字段。作为映射，结构体存储了一个叫做`__struct__`的“特殊”字段，保存了结构体的名字：

```elixir
iex> is_map(john)
true
iex> john.__struct__
User
```

注意我们把结构体成为裸映射是因为它没有实现任何映射的协议。比如，你无法通过枚举来访问一个结构体：

```elixir
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (UndefinedFunctionError) function User.fetch/2 is undefined (User does not implement the Access behaviour)
             User.fetch(%User{age: 27, name: "John"}, :name)
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

然而，由于结构体只是映射，所以可以使用`Map`模块中的函数来操作它：

```elixir
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

结构体和协议为开发者提供了Elixir中最重要的特性：数据多态性。我们将在下一章中讨论它们。

Default values and required keys

## 键的默认值和设置必须指定值的键

如果你在定义一个结构体时没有指定一个键的默认值，这个键的值将被设为`nil`：

```elixir
iex> defmodule Product do
...>   defstruct [:name]
...> end
iex> %Product{}
%Product{name: nil}
```

你可以在创建结构体时强制声明特定的键必须被指定一个值：

```elixir
iex> defmodule Car do
...>   @enforce_keys [:make]
...>   defstruct [:model, :make]
...> end
iex> %Car{}
** (ArgumentError) the following keys must also be given when building struct Car: [:make]
    expanding struct: Car.__struct__/1
```
