--- 
layout: post
typo_id: 77
title: googletest Hello World
---
<p>
This is a quick run down of how to get started with using <a href="http://code.google.com/p/googletest/">googletest</a> on Ubuntu.
</p>

<!--more-->

<h1>Preparation</h1>
<p>
Assuming you have a working GCC build environment, all you have to do is install the googletest packages:
</p>

<pre>
  $ sudo apt-get install libgtest0 libgtest-dev
</pre>

<h1>Makefile</h1>
<p>
The only think of note about the Makefile is that it includes 'libgtest_main' - which implements main() and calls RUN_ALL_TESTS()
</p>

{% codeblock Makefile - Makefile %}
NAME = hello-world

LIBS = -lgtest_main

debug: all
run-debug:
	./${NAME}

all: $(NAME).o
	c++ -lstdc++ $(LIBS) -o $(NAME) $(NAME).o

compile: $(NAME).o

clean:
	find . -name '*.o' -exec rm -f {} ';'
	find . -name $(NAME) -exec rm -f {} ';'

$(NAME).o: $(NAME).c++
	gcc -c -I. -o $(NAME).o $(NAME).c++

.c++.o:
	gcc -c -I. -o $@ $<
{% endcodeblock %}

<h1>Source</h1>
<p>
I've put everything into a single source file to keep things minimal:
</p>

{% codeblock Hello World and Test - hello_world.cpp %}
/////////////////////////////
// In the header file

#include &lt;sstream&gt;
using namespace std;

class Salutation
{
public:
  static string greet(const string& name);
};

///////////////////////////////////////
// In the class implementation file

string Salutation::greet(const string& name) {
  ostringstream s;
  s << "Hello " << name << "!";
  return s.str();
}

///////////////////////////////////////////
// In the test file
#include &lt;gtest/gtest.h&gt;

TEST(SalutationTest, Static) {
  EXPECT_EQ(string("Hello World!"), Salutation::greet("World"));
}
{% endcodeblock %}

<h1>Compilation</h1>
<p>Just run:</p>
<pre>
  $ make
</pre>

<h1>Output</h1>
<p>
This test produces the following:
</p>

<pre>
$ ./hello-world
Running main() from gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from SalutationTest
[ RUN      ] SalutationTest.Static
[       OK ] SalutationTest.Static
[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran.
[  PASSED  ] 1 test.
</pre>

<h1>Conclusion</h1>
<p>
It couldn't really be much simpler!
</p>
