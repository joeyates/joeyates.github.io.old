---
layout: post
title: "Create a Mix Task for an Elixir Project"
date: 2015-07-25 11:36:16 +0200
comments: true
categories: elixir
---
Mix tasks are helper tasks in Elixir projects.

In this post, I'll create an empty project and a "Hello World!"

I can create a new project with
```sh
$ mix new mix_task_example
```

Now, inside the new `mix_task_example` project directory, running

```sh
$ mix help
```

shows which mix tasksi are available and in this newly created project there
are already more than 30.

Now I create the directory for tasks with
```sh
$ mkdir -p lib/mix/tasks
```

and create a file with the name of the task, e.g. `mix_task_example.salute.ex`
with the following contents:
```elixir
defmodule Mix.Tasks.MixTaskExample.Salute do
  use Mix.Task

  def run(_) do
    IO.puts "Hello World!"
  end
end
```

This far, the task is not yet available, I need to compile the project
```
mix compile
```

And now I can run it with
```sh
$ mix mix_task_example.salute
Hello World!
```

The task doesn't show up in the listing with `mix help`.
To make it show up I need to add @shortdoc
```elixir
defmodule Mix.Tasks.MixTaskExample.Salute do
  use Mix.Task

  @shortdoc "Give a short salutation"

  def run(_) do
    IO.puts "Hello World!"
  end
end
```

If the tasks are part of a library, they will be available to project that
include it as a dependency, so it's important to use the library name
(e.g. `MixTaskExample`) in the module name in order to keep tasks in
a collective namesapce.

As a final touch, let's accept a parameter

```elixir
defmodule Mix.Tasks.MixTaskExample.Salute do
  use Mix.Task

  @shortdoc "Give a short salutation"

  def run(name) do
    IO.puts "Hello #{name}!"
  end
end
```

```sh
mix compile
mix mix_task_example.salute Joe
Hello Joe!
```

A working mix task!

The code for this example can be found at https://github.com/joeyates/mix_task_example
