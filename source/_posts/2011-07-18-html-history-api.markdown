--- 
layout: post
typo_id: 102
title: HTML History API
---
{% img /images/html5-history-api.png The Example App %}

<p>
The HTML5 history API allows AJAX-based sites to avoid "breaking the back button". Every time you update the page, you store the new content is the window.history object. When the user presses the back button, you retrieve the current item from window.history and update the page with it.
</p>

<!--more-->

<p>
What follows is an "all-in-one page" Sinatra/jQuery program. You can find the code <a href="https://gist.github.com/1090336">here</a>.
</p>

Run it like this:
{% codeblock run the program - run.sh %}
  $ ruby history_api.rb
{% endcodeblock %}

The program allows you to browse back and forwards through numbered pages.

When the first page is shown, the initial content of the container is stored in the history object:
{% codeblock Replace State - replace_state.js %}
history.replaceState( $( '#container' ).html(), '', location.href );
{% endcodeblock %}

As you select links, only the content of the 'container' DIV gets updated. An AJAX call is made to get the content, which is stored in the history object:
{% codeblock onClick Callback - on_click.js %}
   var anchor = this;
    $.get( this.href, function( data ) {
      history.pushState( data, '', anchor.href );
      $( '#container' ).html( data );
      updateLinks( anchor.getAttribute( 'data-number' ) );
    } );
{% endcodeblock %}

Each time the back button is pressed, the content is retrieved and placed in the container:
{% codeblock onpopstate - onpopstate.js %}
onpopstate = function( event ) {
  $( '#container' ).html( event.state );
};
{% endcodeblock %}

<p>
Notice that pressing the forward and back buttons updates the page correctly without needing to call the server.
</p>
