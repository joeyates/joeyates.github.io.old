--- 
layout: post
typo_id: 5
title: JRuby on Rails
---
# Which Java Version?

JRuby is normally run on the "official" Sun (now Oracle) Java. Some people seem to be able to get it to run on GNU Java (GCJ), but I suggest sticking with Sun Java to get you started.

# Which Type of Installation?

First of all you should decide where to install JRuby - do you want to install it for all system users or just for your own user?

<!--more-->

# Download

Download the latest JRuby from <a href="http://jruby.org/download">http://jruby.org/download</a>.
    $ wget http://jruby.kenai.com/downloads/1.4.0/jruby-bin-1.4.0.tar.gz
    $ tar -xkzf jruby-bin-1.4.0.tar.gz

## Personal Installation
    $ cd ~/bin
    $ cp -r ../jruby-1.4.0 .

Make a symlink to simplify things in the future when we upgrade to a new version:
    $ ln -s jruby-1.4.0 jruby

Add the following line to your ~/.bashrc to add JRuby executables to your %PATH:
    export PATH=~/bin/jruby/bin:$PATH

## System-wide Installation
    $ sudo cp jruby-1.4.0 /usr/local/lib
    $ sudo ln -s /usr/local/lib/jruby-1.4.0 /usr/local/lib/jruby

# Gems

List installed gems:
    $ jruby -S gem list

Install a gem (in a personal installation):
    $ jruby -S gem install rails

Install a gem (in a system-wide installation):
    $ sudo /usr/local/lib/jruby/bin/jruby -S gem install rails

#	Modifying your Rails Project

You need to look at the list of dependencies of your project, principally the gems, and make sure they are available for JRuby.

Gems written in pure Ruby work 'out of the box' with JRuby. But you'll need to find replacecements for gems which are written in whole or in part in C.

One example of this is RMagick (the Ruby interface to the ImageMagick/GraphicsMagick packages). Ruby on Rails which run on MRI Ruby and use RMagick will have to use RMagick4J under JRuby.

If your Ruby on Rails application has no gem dependency problems, you should be able to run it by changing the name of the database adapter:

{% codeblock Dynamic Database Setup According to Ruby Platform  - config/database.yml %}
<% if RUBY_PLATFORM =~ /java/ %>
development:
adapter: jdbcsqlite3
database: db/development.db
<% else %>
development:
adapter: sqlite3
dbfile: db/development.db
<% end %>
{% endcodeblock %}

Here I have made use of the fact that Rails runs config/database.yml through ERB before loading it.

Once the database adapter setting has changed, you should be able to run
    $ ./script/server

Packaging your application for a servlet container can be done with the [warbler gem](http://rubygems.org/gems/warbler).
