--- 
layout: post
typo_id: 101
title: Overriding the Insertion Operator for C++ Template Classes
---
{% codeblock Dump an Active Record object to Standard Out - dump_active_record_object.cpp %}
Person person( 123 );
std::cout << person << endl;
{% endcodeblock %}
While working on my <a href="https://github.com/joeyates/cpp-active-record/">C++ ActiveRecord implementation</a>, I had a few problems implementing the insertion operator for the main ActiveRecord::Base class.

The class ActiveRecord::Base class is a template class, and the problem was how to correctly declare the operator.

At the time I was unable to find examples on the Internet, so I thought I'd provide my own.

<!--more-->

# The Insertion Operator

By "insertion operator" I mean
{% codeblock Insertion Operator - insertion_operator.cpp %}
ostream & operator<<( ostream &out_stream, const C& c );
{% endcodeblock %}

This is a global function that can called like this:
{% codeblock Calling the Insertion Operator - calling_insertion_operator.cpp %}
std::cout << c;
{% endcodeblock %}

This operator allows you to output a string representation of your classes, which is very handy for debugging:

{% codeblock Dumping Objects - dump_object.cpp %}
C c;
c.do_stuff();
std::cout << c << endl;
{% endcodeblock %}

# The Example
## A Class
Here's an example class which (rather reduntantly) wraps an STL std::list:

{% codeblock MyList - my_list.cpp %}
#include <list>

using namespace std;

template < class T >
class MyList {
 public:
  MyList() {};
  void add( T item ) {
    data_.push_back( item );
  }
  int length() const {
    return data_.size();
  }
 private:
  list< T > data_;
};
{% endcodeblock %}

## Serialization
While debugging and testing, it would be handy to be able to output a string representation of an instance of this class.

{% codeblock Using MyList - using_my_list.cpp %}
MyList<int> ints;
ints.add( 42 );
ints.add( 13 );
std::cout << ints << endl;
{% endcodeblock %}

Which should work like this:
    $ ./my_list
    2 items:
      42
      13

## Implementation

Here's an implementation that just outputs the number of items:
{% codeblock Insertion Operator for MyList - my_list_insertion_operator.cpp %}
template< class T >
ostream & operator<<( ostream &out_stream, const MyList< T >& a_list ) {
  out_stream << a_list.length() << " items:" << endl;
  return out_stream;
}
{% endcodeblock %}

Next, we want the operator to iterate over the list&lt;T&gt; member and output its values.

The main sticking point for me here was the declaration of the iterator. The following is not sufficient:

{% codeblock Incorrect MyList Iteration 1 - my_list_iteration1.cpp %}
...
for( list<T>::const_iterator it = a_list.data_.begin(); it != a_list.data_.end(); ++it ) {
...
{% endcodeblock %}

and produces this error message (with gcc):

    to_ostream.cpp: In function 'std::ostream& operator<<(std::ostream&, const MyList<T>&)':
    to_ostream.cpp:7: error: expected `;' before 'it'
    to_ostream.cpp:7: error: 'it' was not declared in this scope

What's missing is the <code>typename</code> keyword to tell the compiler that we're instantiating an iterator:
{% codeblock MyList Iteration - my_list_iteration.cpp %}
...
for( typename list<T>::const_iterator it = a_list.data_.begin(); it != a_list.data_.end(); ++it ) {
...
{% endcodeblock %}

Here's the final implementation:
{% codeblock MyList Insertion Operator - my_list_insertion_operator.cpp %}
template< class T >
ostream & operator<<( ostream &out_stream, const MyList< T >& a_list ) {
  out_stream << a_list.length() << " items:" << endl;
  for( typename list<T>::const_iterator it = a_list.data_.begin(); it != a_list.data_.end(); ++it ) {
    out_stream << "\t" << *it << endl;
  }
  return out_stream;
}
{% endcodeblock %}

The function also has to access the private `data_` member, which entails declaring it as a `friend`:
{% codeblock Making the Insertion Operator a Friend - insertion_operator_friend.cpp %}
template < class T >
class MyList {
 template< class T1 >
 friend ostream & operator<<( ostream &out_stream, const MyList< T1 >& a_list );
 ...
};
{% endcodeblock %}

Here's the whole class with the operator:
{% codeblock MyList - my_list.cpp %}
#include <iostream>
#include <list>

using namespace std;

template < class T >
class MyList {
 template< class T1 >
 friend ostream & operator<<( ostream &out_stream, const MyList< T1 >& a_list );
 public:
  void add( T item ) {
    data_.push_back( item );
  }
  int length() const {
    return data_.size();
  }
 private:
  list< T > data_;
};

template< class T >
ostream & operator<<( ostream &out_stream, const MyList< T >& a_list ) {
  out_stream << a_list.length() << " items:" << endl;
  for( typename list<T>::const_iterator it = a_list.data_.begin(); it != a_list.data_.end(); ++it ) {
    out_stream << "\t" << *it << endl;
  }
  return out_stream;
}
{% endcodeblock %}

And here's an example of it's use:
{% codeblock Using MyList - using_my_list.cpp %}
#include <string>

int main( int argc, char **argv ) {
  MyList< int > my_ints;
  my_ints.add( 1 );
  my_ints.add( 2 );
  
  cout << "myints" << endl;
  cout << my_ints << endl;

  MyList< string > my_strings;
  my_strings.add( "hello" );
  my_strings.add( "world" );

  cout << "mystrings" << endl;
  cout << my_strings << endl;

  return 0;
}
{% endcodeblock %}

The output:
    $ ./to_ostream 
    myints
    2 items:
            1
            2

    mystrings
    2 items:
            hello
            world
