---
layout: post
title: "How to forward blocks in Ruby"
date: 2014-06-30 09:51:19 +0200
comments: true
categories: [Ruby]
---

# TL;DR

Use `Proc.new`

# Calling Enumerators - normal use

You're writing some code which calls an Enumerator - a function that
makes repeated calls to the block of code that you provide.

```ruby
def yield_me_2_things
  yield 'Thing 1'
  yield 'Thing 2'
end

yield_me_2_things { |x| puts x }
```

This will print:

```
Thing 1
Thing 2
```

The values are supplied by `yield_me_2_things` and the printing is done in
the block, `{ |x| puts }`, that is passed to that method.

# Generalize

I can now make a generalized method, to handle any number of things:

```ruby
def yield_me_n_things(n)
  1.upto(n) do |i|
    thing = "Thing #{i}"
    yield thing
  end
end

yield_me_n_things(2) { |x| puts x }
```

...the output is the same.

# An alternative: use a block

I could equally have implemented the method using a `&block` parameter -
for the caller, it makes no difference:

```ruby
def call_this_block_with_n_things(n, &block)
  1.upto(n) do |i|
    thing = "Thing #{i}"
    block.call thing
  end
end

call_this_block_with_n_things(2) { |x| puts x }
```

...the output is the same.

# The problem

What if I want one Enumerator to call another?

What if I want to keep the specific version (`yield_me_2_things`)
but just make it call the generalized method?

```ruby
def enumerate_n_things(n) # How do I receive the block?
  1.upto(n) do |i|
    thing = "Thing #{i}"
    # How do I call the block?
  end
end

def enumerate_2_things
  enumerate_n_things(2) # How do I forward the block?
end

enumerate_2_things { |x| puts x }
enumerate_2_things(2) { |x| puts x }
```

How should I write the two methods, while keeping both usable indipendently?

# Attempt 1: Forward using yield

With `yield`, you don't explicitly receive the block, you just call it.
Does that work across two levels? I.e., does the block get passed to method I call?

```
def enumerate_n_things(n)
  1.upto(n) do |i|
    thing = "Thing #{i}"
    yield thing
  end
end

def enumerate_2_things
  enumerate_n_things(2)
end

enumerate_2_things { |x| puts x }
```

No, doesn't work, `enumerate_n_things` doesn't receive a block.

I get this error:
```
no block given (yield) (LocalJumpError)
```

# Attempt 2: Forward using a block

```
def enumerate_n_things(n, block) # Note: no '&'
  1.upto(n) do |i|
    thing = "Thing #{i}"
    block.call thing
  end
end

def enumerate_2_things(&block)
  enumerate_n_things(2, block)
end

enumerate_2_things { |x| puts x }
```
Prints:
```
Thing 1
Thing 2
```
But we can no longer pass a block to the generalized method:
```
enumerate_n_things(2) { |x| puts x }
```

`enumerate_n_things` now expects the block as a normal parameter.

I get this error:
```
wrong number of arguments (1 for 2)
```

# Solution: Proc.new

```ruby
def enumerate_n_things(n, block = Proc.new)
  1.upto(n) do |i|
    thing = "Thing #{i}"
    block.call thing
  end
end

def enumerate_2_things(block = Proc.new)
  enumerate_n_things(2, block)
end

enumerate_2_things { |x| puts x }
enumerate_n_things(2) { |x| puts x }
```
Both calls now work!

`Proc.new` transforms any block passed to a method into a Proc.
If we use that as the default value for a block parameter we can
call methods directly with blocks, or forward blocks between
enumerators.
