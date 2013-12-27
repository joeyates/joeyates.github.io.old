--- 
layout: post
typo_id: 42
title: i18n Internationalization without Rails
---
I searched the Internet in vain for examples of using the Ruby [i18n gem](http://rubygems.org/gems/i18n) outside of Rails apps.

<!--more-->

Below is a minimal example: the aim is (as usual) to produce a greeting - this time in English and Italian.

# Installation
    $ gem install i18n

# The `t` method

The most common use of i18n is via `I18n.translate` which has an abbreviated alias `I18n.t`.

#	Translations

{% codeblock English - en.yml %}
en:
  hello world: Hello World!
{% endcodeblock %}

{% codeblock Italian - it.yml %}
it:
  hello world: Ciao Mondo!
{% endcodeblock %}

These translations can be accessed as follows:
    I18n.t( 'hello world' )

That will use the current locale setting. You can change it as follows:
    I18n.locale = :it

#	The Program

After loading the `i18n` gem, set up `I18n.load_path` before attempting to retrieve any transaltions.

{% codeblock Use the Translations - salutation.rb %}
require 'rubygems' if RUBY_VERSION < '1.9'
require 'i18n'

I18n.load_path = ['en.yml', 'it.yml']

puts 'In English...'
I18n.locale = :en
puts I18n.t('hello world')

puts 'In Italian...'
I18n.locale = :it
puts I18n.t('hello world')
{% endcodeblock %}

The result:

    $ ruby salutation.rb
    In English...
    Hello World!
    In Italian...
    Ciao Mondo!
