title: Elixir Pattern Matching Struct And Map
date: 2017-03-23 10:56:12
tags:
- Elixir
- Learning
---

I started learning [Elixir][1] recently, while working on a personal project I had to pattern match between struct and map in a function. While glancing through the documentation I came across a function `is_map`, as the name suggests it check if the argument is map or not. Coming from a C# background I thought it is going to work, but it didn't.

In [Elixir][1] struct is a special kind of map, so `is_map` function matches both. Then I went through some of the open source code bases and came across a way to do this(don't remember from which project I saw this)

```Elixir
# This match struct
def clean_up(data = %_{}) do
  .....
end

# This match map
def clean_up(data = %{}) do
  ....
end
```

Very powerful pattern matching technique and elegant solution. Here if `data` is a struct it matches first function and if it is map matches the second one.

[1]: http://elixir-lang.org/
