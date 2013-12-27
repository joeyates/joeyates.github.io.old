--- 
layout: post
typo_id: 80
title: "Sinatra: A Real Site"
---
{% img /images/sidcom.png sidcom.me.uk %}

<h1>The Site</h1>
<p>
I have a young relative who draws a lot of comic strips, and thinking it would be nice if he could publish them, I looked around for an Open Source system.<br>
I wanted something that was as near to XKCD as possible - latest comic, next/previous and random.<br>
Not finding anything that was both pretty and ultra-simple, I decided to write my own.<br>
The version for my relative is <a href="http://sidcom.me.uk/">here</a> -
the system will be up on GitHub as soon as I finish making it configurable.<br>
</p>

<!--more-->

<h1>Technologies</h1>
<h2>Web Framework</h2>
<p>
If you've read the title of this post, you won't be surprised to read that I chose <a href="http://www.sinatrarb.com/intro.html">Sinatra</a>.<br>
</p>

<h2>Database</h2>
<p>
I decided to use <a href="http://datamapper.org/">DataMapper</a> as the ORM.<br>
I'm also using SQLite for both development and production.<br>
</p>

<h2>HTML and CSS</h2>
<p>
As will be clear from the example site, I didn't work over hard on making a fresh graphic look -
all I did was strip out a lot of HTML that was there to make rounded corners,
and replaced it with Mozilla and WebKit specific rounding.<br>
I then translated the HTML into <a href="http://haml-lang.com/">HAML</a> and
the CSS into <a href="http://sass-lang.com/">SASS</a> (SCSS).<br>
</p>

<h1>Development</h1>
<p>
Documentation for Sinatra, DataMapper, HAML and SASS is quite good, so things went quite smoothly.
What follows is a list of the phases I went through while developing (on Ubuntu), and especially the bits I had trouble with.<br>
</p>

<h1>DataMapper</h1>
<h2>Installation</h2>
<pre>
  $ sudo gem install dm-core

  $ sudo gem install data_objects
  $ sudo gem install data_mapper
  $ sudo gem install do_sqlite3
</pre>

<h2>Using DataMapper in Code</h2>

{% codeblock DataMapper Setup - comic.rb %}
  require 'dm-core'
  require 'data_mapper'

  DataMapper.setup({
    :adapter => 'sqlite3',
    :host => 'localhost',
    :username => '',
    :password => '',
    :database => 'db/sidcom_development'
  })

  class Comic
    include DataMapper::Resource
  
    property :title, :text
    property :permalink, :text
    property :created_at, :datetime
    property :updated_at, :datetime
  end
  Comic.auto_upgrade!
{% endcodeblock %}

The '<code>Comic.auto_upgrade!</code>' bit handles migrations -
at startup, table columns are adjusted to match the declaration in the equivalent class.

<h1>Rack</h1>
<h2>Authentication</h2>
<p>
This application has a 'public' and an 'admin' area, so I actually created two Sinatra Apps,
wrapping one in <a href="http://rack.rubyforge.org/doc/Rack/Auth/Digest/MD5.html">Rack::Auth::Digest::MD5</a>.
A local settings file holds user names and hashed (kinda salted) passwords.<br>
</p>

<h1>Sinatra</h1>
<p>
Sinatra apps have access to a number of <a href="http://yardoc.org/docs/sinatra-sinatra/Sinatra/Base">globals</a>, I used:<br>
<ul>
  <li>
    <a href="http://rack.rubyforge.org/doc/classes/Rack/Request.html">request</a>,
  </li>
  <li>
    params<br>
  </li>
</ul>
</p>

<h2>Helpers</h2>
<p>
This app being a little more complex than my first, I decided to implement some Rails-like helpers.<br>
The most important helper is <code>url_for</code>.
Sinatra - by design - includes routing in the structure of an app, so it cannot provide pre-cooked methods of the sort.<br>
</p>

<h1>HAML</h1>
<p>
I found HAML very easy to start using, what I didn't immediately understand was:<br>
<ul>
  <li>
    how to interpolate variables,
  </li>
  <li>
    how to call functions,
  </li>
  <li>
    and how to include partials.
  </li>
</ul>
</p>
<p>
These are actually all the same problem, but it took me a while to realize that!<br>
In HAML, you can interpolate the Ruby way (with '<code>#{...}</code>') or the HAML way (with '<code>=</code>').<br>
</p>

<h2>Variable Interpolation</h2>
<p>
Just use Ruby String interpolation:<br>
</p>
<pre>
  %h1 Hello #{ @name }!
</pre>

<h2>Calling Functions</h2>
<p>
Use HAML evaluation syntax:<br>
</p>
<pre>
  %h1
    = @name
    !
</pre>

<h2>HAML partials</h2>
<p>
Again, these can be evaluated as above:<br>
</p>
<pre>
  = haml(:_my_partial, :layout =&gt; false)
</pre>

<h1>SASS</h1>
<h2>Generating CSS</h2>
<p>
  I chose to keep my SCSS files in a subdirectory under my views directory, and generate CSS from them under the static path.
</p>
<p>
During development, you need to keep the static versions up to date, and to do so, you just need to keep a process running as follows:<br>
</p>
<pre>
  $ sass --watch views/stylesheets:static/stylesheets
</pre>

<h1>Conclusion</h1>
<p>
Sinatra handles small sites very well.
The code can be kept DRY, and if you decide to, Sinatra projects can be quite easily transformed into Rails projects.<br>
Actually, I have doubts whether it was worth doing this project in Sinatra -
a stripped-down Rails 3 project would have been just as lightweight and easy to develop.
But, then again, it's useful to run a sytem through its paces, to have an idea what areas it can usefully be applied to.<br>
</p>
