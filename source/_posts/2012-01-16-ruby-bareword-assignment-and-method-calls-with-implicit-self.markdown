---
layout: post
title: "Ruby Bareword Assignment and Method Calls With Implicit Self"
date: 2012-01-16 18:52
comments: true
categories: ruby
---
Problem
=======

If I do this:

{% codeblock Problem - problem.rb %}
puts foo
foo = 3
{% endcodeblock %}

there is always the doubt whether I'm accessing a local variable, or calling methods `foo` and `foo=`.

TL;DR
=====

When you want to call an instance's own methods, use `self`:

{% codeblock TL;DR - tldr.rb %}
self.foo             # Calls foo
self.foo = 'bar'     # Calls foo=
{% endcodeblock %}

<!--more-->

Example 1
=========

{% codeblock Example 1 - example_1.rb %}
def example1
 'example1 method'
end

example1 #=> "example1 method"

example1 = 'assigned value'

example1 #=> "assigned value"
{% endcodeblock %}

Here, we define a method, and then make an assignment. As we assign to a bareword, Ruby creates a new local variable.

As soon as a value is assigned to the local variable, the method no longer gets called.

Example 2
=========

But, what if we also have an assignment method?

{% codeblock Example 2 - example_2.rb %}
def example2
 'example2 method'
end

def example2=(value)
 puts "example2= called" # (this never gets called)
end

example2 #=> "example2 method"

example2 = 'assigned value'

example2 #=> "assigned value"
{% endcodeblock %}

Adding the method `example2=` does not change things. When we assign to a bareword, Ruby takes it as assignment to a local variable.

Example with a Class
====================

{% codeblock x - x.rb %}
class Foo

 attr_accessor :bar

 def initialize
   @bar = 42
 end

 def method1
   puts bar
 end

 def method2
   bar = 99
   puts bar
 end

 def method3
   bar = 99
   puts self.bar
 end

end

foo = Foo.new

foo.bar  #=> 42
foo.method1 #=> 42
foo.method2 #=> 99
foo.method3 #=> 42
{% endcodeblock %}

`method2` is the problem case. `bar` is assigned to, creating a local variable, so subsequent calls to `bar` return 99.
`method3` disambiguates by explicitly calling the `bar` method on `self`.

The Cause
=========

There are two things going on here:

1. bareword assignment creates local variables,
2. local variables mask methods of the same name.

Refactoring Might Break Code
============================

One solution is to use `self.method` only in cases where local variables mask methods. The problem with this approach is that code may be altered, introducing local variables, and so altering the behaviour of following code:

Original Code
-------------

{% codeblock x - x.rb %}
class Foo
 attr_accessor :bar

 def baz
   puts bar
 end
end

foo = Foo.new
foo.bar = 42
foo.baz #=> 42
{% endcodeblock %}

Modified Code
-------------

{% codeblock x - x.rb %}
class Foo
  attr_accessor :bar

  def baz
    bar = 99 # <= variable assignment introduced
    puts bar
  end
end

foo = Foo.new
foo.bar = 42
foo.baz #=> 99
{% endcodeblock %}

Solution
========

The best solution is to always call instance methods on `self`.
