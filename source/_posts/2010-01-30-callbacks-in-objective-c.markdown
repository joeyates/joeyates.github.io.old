---
layout: post
typo_id: 4
title: Callbacks in Objective-C
---
I've been learning Objective-C and was implementing a regular expression library on top of libpcre. I wanted to add a way of accepting a user-defined function in replace operations, similar to Ruby's `String.gsub()` or Perl `s///e`.

I realized that I needed to handle callbacks, but in Objective-C, you're not supposed to play with function pointers.

In C or C++ the pattern is very common, but I was initially stumped when I had to implement the same pattern in Objective-C. All the search results I got seemed to say you couldn't do it.

<!--more-->

# The C Way

In C you pass the callback function as a function pointer:

{% codeblock Example of a C Callback - c_style.c %}
#include <stdio.h>
#include <string.h>

void caller(void (* callback)(const char *))
{
  printf("'caller' invoking callback\n");
  callback("Hello C");
}

void my_callback(const char * sz)
{
  printf("'my_callback' called with greeting '%s'\n", sz);
}

int main()
{
  caller(my_callback);
  return 0;
}
{% endcodeblock %}

Which results in:

    $ ./callback
    'caller' invoking callback
    'my_callback' called with greeting 'Hello C'

# Selector From String

In Objective-C you don't actually call methods. You send messages to objects, and the Objective-C runtime invokes the correct method by matching the message signature with available selectors for that object.

What follows is GNUStep-specific, but should work with Apple's Objective-C method `NSSelectorFromString`.

# An Objective-C Equivalent

Here's the object we want to call back to, it has a method which takes one parameter:

{% codeblock The Class to Call Back To - responder.m %}
@interface Responder : NSObject
  - (id) my_callback: (NSString *) sGreeting;
@end

@implementation Responder
- (id) my_callback: (NSString *) sGreeting
{
  printf("'Responder.my_callback' called with greeting: '%s'.\n", [sGreeting cString]);
}
@end
{% endcodeblock %}

Here's the calling code.

{% codeblock The Calling Class - caller.m %}
@interface Caller : NSObject
+ (void) call: (id) obj method: (const char *) szMethod;
@end

@implementation Caller
+ (void) call: (id) obj method: (const char *) szMethod
{
  SEL sel = GSSelectorFromName(szMethod);
  printf("Caller invoking method '%s'\n", szMethod);
  [obj performSelector: sel withObject: @"Hello Objective-C"];
}
@end
{% endcodeblock %}

The name of the method we want to call is `my_callback:` - note the `:`.

{% codeblock main - main.m %}
int main(void)
{
  Responder * res = [[Responder alloc] init];
  [Caller call: res method: "my_callback:"];
  [res dealloc];
}
{% endcodeblock %}

The result of all this is:

    $ ./callback
    Caller invoking method 'my_callback:'
    'Responder.my_callback' called with greeting: 'Hello Objective-C'.

# Conclusion

This seems rather long-winded, but in the end most of the extra lines of code here relate to the fact we're using objects, not plain functions as we do in C.

The next step would be to benchmark this to get an idea how much time the Objective-C runtime takes to do the method lookup with `GSSelectorFromName` and to carry out the call.
