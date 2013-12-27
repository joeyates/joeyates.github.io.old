--- 
layout: post
typo_id: 100
title: Hello Gosu!
---
{% img /images/hello_gosu.png Hello Gosu! %}

I'm trying out <a href="http://www.libgosu.org/">Gosu</a>, which is a 2D game library written in C++ with Ruby bindings (<a href="https://secure.wikimedia.org/wikipedia/en/wiki/Gosu_%28library%29">Wikipedia article</a>).

<!--more-->

The <a href="http://www.libgosu.org/rdoc/files/README_txt.html">Ruby documentation</a> is sparse but sufficient.

The gem is easy to install:
    $ gem install gosu

A 'Hello World!' app requires about 20 lines of code:<br />
{% codeblock Hello World in Gosu - hello_world.rb %}
require 'rubygems'
require 'gosu'

class HelloWindow < Gosu::Window

  def initialize
    super( 250, 100, false )
    @background_image = Gosu::Image.new( self, 'hello_world.png' )
  end

  def draw
    @background_image.draw( 0, 0, 1 )
  end

  def button_down(id)
    close
  end

end

HelloWindow.new.show
{% endcodeblock %}

Run it like this:
    $ ruby hello_gosu.rb

The program creates the window shown at the start of this article.

The complete code is in a [GitHub repository](https://github.com/joeyates/hello_gosu).
