--- 
layout: post
typo_id: 58
title: A Beginner's Sinatra Tutorial
---
In the olden days your first program was probably in BASIC and was run from the command line, nowadays the equivalent is Web programming.

Using Sinatra reminds me of my first programming experiences - you write a tiny bit of code, with no boilerplate and get to see a result.

<!--more-->

What follows owes a lot to the <a href="http://www.sinatrarb.com/intro.html">Sinatra README</a>.

# Installation

On Linux, you'll need Ruby and Rubygems installed (if not you should find installation instructions easily enough via a Web search).

Apart from Ruby, you'll need to use a text editor (e.g. GEdit) a Web browser and the Terminal (normally found in the Applications menu under Accessories).

Next install Sinatra:
    $ gem install sinatra

# 1. Hello World!

{% codeblock A Minimal Sinatra App - minimal.rb %}
require 'rubygems' if RUBY_VERSION < '1.9'
require 'sinatra';

get '/' do
  'Minimal Sinatra Hello World!'
end
{% endcodeblock %}

Now, open a Terminal window and type:
    ruby minimal.rb

Finally, go to this address: <a href="http://127.0.0.1:4567/">http://127.0.0.1:4567/</a>

To stop the program, go to the Terminal window and press Ctrl+C

### 2. Adding Structure: Sinatra::Base

It's useful to structure the program as a class.

{% codeblock Sinatra::Base - sinatra_base.rb %}
require 'rubygems' if RUBY_VERSION < "1.9"
require 'sinatra/base'

class MyApp < Sinatra::Base
  get '/' do
    'Hello World!'
  end
end

MyApp.run!
{% endcodeblock %}

Run it:
    ruby sinatra_base.rb

### 3. Structure: A Separate App File

Create two files:

{% codeblock Application - my_app.rb %}
require 'rubygems' if RUBY_VERSION < "1.9"
require 'sinatra/base'

class MyApp < Sinatra::Base
  get '/' do
    'Hello World from MyApp in separate file!'
  end
end
{% endcodeblock %}

{% codeblock Launcher - run_my_app.rb %}
require 'my_app'

MyApp.run!
{% endcodeblock %}

Run it:
    $ ruby run_my_app.rb

# 4. Rackup

Instead of writing a script to start the app, we can use Rackup to launch the same script:

Create a file 'config.ru':
    require 'my_app'

    run MyApp.new

Run it:
    rackup config.ru

This time the application is visible at <a href="http://127.0.0.1:9292">http://127.0.0.1:9292</a>

If we want it to have the same address as before, we can run it like this:
    rackup config.ru --port=4567

### 5. Static Files

A Web application will often have dynamic content (what we've created so far) and static content (e.g. images).

Create a directory 'static', and create a file called index.html in containing:

This is a static file.

{% codeblock Static Files - my_app.rb %}
require 'rubygems' if RUBY_VERSION < "1.9"
require 'sinatra/base'

class MyApp < Sinatra::Base
  set :static, true
  set :public, File.dirname(__FILE__) + '/static'
end
{% endcodeblock %}

Now, if you run rackup, and go to <a href="http://127.0.0.1:4567/index.html">http://127.0.0.1:4567/index.html</a> you will see the content.

# 6. Routes

{% codeblock English - static/routes.html %}
<ul>
  <li><a href="named_via_params/foo">Named parameters via params Hash</a></li>
  <li><a href="named_via_block_parameter/foo">Named parameters via block parameters</a></li>
  <li><a href="splat/foo/bar/baz">Multiple parameters via params[:splat]</a></li>
  <li><a href="splat_extension/myfile.txt">Filename via params[:splat]</a></li>
  <li><a href="regexp_params_captures/foobar">Using regular expression via params[:captures]</a></li>
  <li><a href="regexp_captures_via_block_parameter/foobar">Using regular expression via block parameters</a></li>
</ul>
{% endcodeblock %}

{% codeblock English - route_examples.rb %}
require 'rubygems' if RUBY_VERSION < "1.9"
require 'sinatra/base'

class MyApp < Sinatra::Base
  set :static, true
  set :public, File.dirname(__FILE__) + '/public'

  get '/named_via_params/:argument' do
    "
Using: '/named_via_params/:argument'<br/>
params[:argument] -> #{params[:argument]} (Try changing it)
"
  end

  get '/named_via_block_parameter/:argument' do |argument|
    "
Using: '/named_via_block_parameter/:argument'<br/>
argument -> #{argument}
"
  end

  get '/splat/*/bar/*' do
    "
Using: '/splat/*/bar/*'<br/>
params[:splat] -> #{params[:splat].join(', ')}
"
  end

  get '/splat_extension/*.*' do
    "
Using: '/splat_extension/*.*'<br/>
filename -> #{params[:splat][0]}<br/>
extension -> #{params[:splat][1]}
"
  end

  get %r{/regexp_params_captures/([\w]+)} do
    "params[:captures].first -> '#{params[:captures].first}'"
  end

  get %r{/regexp_captures_via_block_parameter/([\w]+)} do |c|
    "c -> '#{c}'"
  end

end
{% endcodeblock %}

Go to <a href="http://127.0.0.1:4567/routes.html">http://127.0.0.1:4567/routes.html</a>

You will see a list of links which exemplify different types of route matching.

# 7. Templates

Create a 'views' directory.

{% codeblock The template - views/index.erb %}
This is <%= @foo %>
{% endcodeblock %}

{% codeblock Templates - templates.rb %}
require 'rubygems' if RUBY_VERSION < "1.9"
require 'sinatra/base'
require 'erb'

class MyApp < Sinatra::Base
  get '/template' do
    @foo = 'erb'
    erb :index
  end
end
{% endcodeblock %}

You should get the following on <a href="http://localhost:4567/template">http://localhost:4567/template</a>:<br />

{% img /images/sinatra-erb-template.png Sinatra ERB Template %}
