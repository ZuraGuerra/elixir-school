---
layout: page
title: Erlang Interoperability
category: advanced
order: 1
lang: en
---


Máquina virtual de Erlang (ErlangVM, en adelante)


One of the added benefits to building on top of the ErlangVM is the plethora of existing libraries available to us.  Interoperability allows us to leverage those libraries and the Erlang standard lib from our Elixir code.  In this lesson we'll look at how to access functionality in the standard lib along with third-party Erlang packages.

## Table of Contents

- [Standard Library](#standard-library)
- [Erlang Packages](#erlang-packages)
- [Notable Differences](#notable-differences)
  - [Atoms](#atoms)
  - [Strings](#strings)
  - [Variables](#variables)


## Standard Library

Erlang's extensive standard library can be accessed from any Elixir code in our application.  Erlang modules are represented by lowercase atoms such as `:os` and `:timer`.

Let's use `:timer.tc` to time execution of a given function:

```elixir
defmodule Example do
  def timed(fun, args) do
    {time, result} = :timer.tc(fun, args)
    IO.puts "Time: #{time}ms"
    IO.puts "Result: #{result}"
  end
end

iex> Example.timed(fn (n) -> (n * n) * n end, [100])
Time: 8ms
Result: 1000000
```

For a complete list of the modules available, see the [Erlang Reference Manual](http://erlang.org/doc/apps/stdlib/).

## Erlang Packages

In a prior lesson we covered Mix and managing our dependencies, including Erlang libraries works the same way.  In the event the Erlang library has not been pushed to [Hex](hex.pm) you can refer the git repo instead:

```elixir
def deps do
  [{:png, github: "yuce/png"}]
end
```

Now we can access our Erlang library:

```elixir
png = :png.create(#{:size => {30, 30},
                    :mode => {:indexed, 8},
                    :file => file,
                    :palette => palette}),
```

## Notable Differences

Now that we know how to use Erlang we should cover some of the gotchas that come with Erlang interoperability.

### Atoms

Erlang atoms look much like their Elixir counterparts without the colon (`:`).  They're they are represented by lowercase strings and underscores:

Elixir:

```elixir
:example
```

Erlang:

```erlang
example.
```

### Strings

Erlang strings are represented by single quotes (`''`) much like Elixir's char lists.  In addition to sharing the syntax with Erlang strings, char lists are more commonly use when interfacing with Erlang code.

Elixir:

```elixir
"Example String"
```

Erlang:

```erlang
'Example String'.
```

It's important to note that many older Erlang libraries may not support binaries so we need to convert Elixir strings to char lists.  Thankfully this is easy to accomplish with the `to_char_list/1` function:

```elixir
iex> :string.words("Hello World")
** (FunctionClauseError) no function clause matching in :string.strip_left/2
    (stdlib) string.erl:380: :string.strip_left("Hello World", 32)
    (stdlib) string.erl:378: :string.strip/3
    (stdlib) string.erl:316: :string.words/2

iex> "Hello World" |> to_char_list |> :string.words
2
```

### Variables

Elixir:

```elixir
iex> x = 10
10

iex> x1 = x + 10
11
```

Erlang:

```erlang
1> X = 10.
10

2> X1 = X + 1.
11
```

That's it!  Leveraging Erlang from within our Elixir applications is easy and effectively doubles the number of libraries available to us.
