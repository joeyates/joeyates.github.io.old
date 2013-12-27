--- 
layout: post
typo_id: 46
title: A Sunrise and Sunset Time Calculator
---
I've released the [ruby-sun-times gem](http://rubygems.org/gems/ruby-sun-times), which calculates sunrise and sunset times according to date and location.

<!--more-->

# The Project

The project is hosted on [GitHub](http://github.com/joeyates/ruby-sun-times).

The gem is available from [here](http://rubygems.org/gems/ruby-sun-times).

# Using ruby-sun-times

    $ irb
    >> require 'rubygems'
    => true
    >> require 'sun_times'
    => true
    >> SunTimes.set( Date.today, 44, 11 ).localtime
    => Tue Mar 09 18:14:08 +0100 2010

So, the sun will be setting round here at 6:14 PM local time.
