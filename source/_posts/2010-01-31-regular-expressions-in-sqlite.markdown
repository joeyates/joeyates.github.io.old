--- 
layout: post
typo_id: 12
title: Regular Expressions in SQLite with Ruby
---
Recently, I was writing an autocomplete method in a Ruby on Rails application. I wanted to find whole word matches, and words starting with the entered text.

The regex I wanted to use was like this:

    word =~ /\W#{query}/

So, if the user entered *'and'*, I wanted to retrieve *'**And**es'* and *'Bill **and** Ben'*, but not *'c**and**le'*.

<!--more-->

In development I was using an SQLite database, and SQLite does not yet implement a regular expression operator. Actually, it *defines* the `REGEX` operator, but it doesn't implement it - if you use it you get an error.

I tried writing the method with just the `LIKE` operator, but it was getting very long-winded: you have to jump through hoops to approximate the regex `\W` operator.

The `REGEXP` operator is just syntactic sugar for the (unimplemented) [regexp() function](http://www.sqlite.org/lang_expr.html#regexp). SQLite allows you to add external functions at runtime, so I realized that there must be a way around the limitation, but, initially I thought I had to implement regexp() as a C function.

I found [an article about implementing regexp() in Ruby](http://stephen-veit.blogspot.com/2009/03/implementing-regexp-in-sqlite3.html). It needed a bit of tweaking as in the Rails 2.3.x interface for SQLite `ActiveRecord::ConnectionAdapters::SQLite3Adapter.initialize()` takes 3 parameters, not 2.

I got `REGEXP` working by creating an initializer:

{% codeblock Implement SQLite's REGEXP Operator in Ruby - config/initializers/sqlite3_regexp.rb %}
require 'active_record/connection_adapters/sqlite3_adapter'

class ActiveRecord::ConnectionAdapters::SQLite3Adapter
  def initialize(db, logger, config)
    super
    db.create_function('regexp', 2) do |func, pattern, expression|
       regexp = Regexp.new(pattern.to_s, Regexp::IGNORECASE)
       if expression.to_s.match(regexp)
         func.result = 1
       else
         func.result = 0
       end
     end
  end
end
{% endcodeblock %}

The proof of the pudding:

    ./script/console
    >> Noun.find(:all, :conditions => ['name LIKE ?', '%and%']).collect(&:name)
    => ["Candle", "Bill and Ben", "Andes"]
    >> Noun.find(:all, :conditions => ['name REGEXP ?', '\Wand']).collect(&:name)
    => ["Bill and Ben", "Andes"]
