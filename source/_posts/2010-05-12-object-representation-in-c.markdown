--- 
layout: post
typo_id: 74
title: Object Representation in C++
---
# Member variables

Each C++ object needs to hold its instance data. But, there is often not a perfect match between the sum of the byte sizes of the member variables and the size of an object: The space occupied will often be greater than the sum of the sizes of the member variables.

The reasons are:
<ul>
<li>compiler-specific alignment issues: in order to speed up access, a few bytes can be wasted for each instance to ensure that all the members are aligned on a word boundary,</li>
<li>the presence of a pointer to the virtual function table,</li>
<li>empty classes.</li>
</ul>

<!--more-->

## An empty class

An empty class has no member variables, but does not occupy 0 bytes of memory. The ISO standard mandates that objects of any class must occupy at least one byte.

To show this, I'll call the following code (see the end for the code used to dump objects):

{% codeblock An Empty Class - empty_class.cpp %}
class A {};

A a;
dump_instance("An empty class:", a);
{% endcodeblock %}

The output shows the memory location of the instance, its size end content:<br />
GCC:<br />
<pre>
An empty class:
  bfd3dfdf -> instance (1 byte):
    00
</pre>
Visual C++:<br />
<pre>
An empty class:
  0012ff5f -> instance (1 byte):
    00
</pre>
</p>
<p>
In both cases, the instance of the empty class occupies 1 byte. And, in both cases, the byte seems to have been initialized to 0.
</p>
<h1>Classes with member variables</h1>
<p>
Next, I'll do the same for a non-empty class with a single <code>int</code> member variable.
(I'll initialize member variables to make the dumps easier to read.)
</p>

{% codeblock One Member Variable - member_variable.cpp %}
class B {
public:
  B(): i(0xbbbbbbbb) {};
private:
  int i;
};

B b;
dump_instance("A class with an int member:", b);
{% endcodeblock %}

The memory occupied is exactly the size of the member (4 bytes):<br />
GCC:<br />
<pre>
A class with an int member:
  bfa25e08 -> instance (4 bytes):
    bb bb bb bb 
</pre>
Visual C++:<br />
<pre>
A class with an int member:
  0012ff64 -> instance (4 bytes):
    bb bb bb bb 
</pre>
</p>
<p>
This case is the simplest of all. You get exactly what you expect: a four byte type occupying 4 bytes.
</p>
<h2>Word alignment</h2>
<p>
char, a single-byte-sized type, gets word-aligned both by GCC and Visual C++. Notice that three uninitilised bytes follow the byte containing 0xaa:
</p>

{% codeblock Word Alignment - word-alignment.cpp %}
class C {
public:
  C(): ch(0xaa), i(0xbbbbbbbb) {};
private:
  char ch;
  int i;
};

C c;
dump_instance("A class with char and int members:", c);
{% endcodeblock %}

GCC:<br />
<pre>
A class with char and int members:
  bfd3dfcc -> instance (8 bytes):
    aa 5e a2 bf bb bb bb bb 
</pre>
Visual C++:<br />
<pre>
A class with char and int members:
  0012ff68 -> instance (8 bytes):
    aa 77 40 00 bb bb bb bb 
</pre>
</p>
<p>
Both implementations allocate 8 bytes, wasting 3 bytes after the char, in order to word align the int.
</p>
<p>
The same is true if the char follows the int:
</p>

{% codeblock Word Alignment - word-alignment.cpp %}
class D {
public:
  D(): i(0xbbbbbbbb), ch(0xaa) {};
private:
  int i;
  char ch;
};

D d;
dump_instance("A class with int and char members:", d);
{% endcodeblock %}

GCC:<br />
<pre>
A class with int and char members:
  bfd3dfc4 -> instance (8 bytes):
    bb bb bb bb aa ad 04 08 
</pre>
Visual C++:<br />
<pre>
A class with int and char members:
  0012ff70 -> instance (8 bytes):
    bb bb bb bb aa bf 40 00 
</pre>
</p>
<h1>
	Member functions</h1>
<p>
	Beyond data, the compiler needs to be able to call the object&#39;s functions. Member functions are shared between all instances of a class and so are only stored in one place.
In the case of non-virtual member functions, nothing needs to be included in the class to allow name resolution.
</p>

{% codeblock Member Functions - member_functions.cpp %}
class NonVirtual {
  void f() {}
};
NonVirtual nv;
dump_instance("A class with a single member function:", nv);
{% endcodeblock %}

GCC:<br />
<pre>
A class with a single member function:
  bfbdeb5e -> instance (1 byte):
    53 
</pre>
Visual C++:<br />
<pre>
A class with a single member function:
  0012ff5f -> instance (1 byte):
    00 
</pre>
</p>
<p>
This example is similar to the empty class above: having no member variables, and no virtual member functions (below), the compiler is forced to waste one byte, to comply with the standard's mandate of not having zero-sized classes. Visual C++ seems to initialize the byte.
</p>
<h2>
	Virtual member functions</h2>
<p>
	Object derivation means that the list member functions callable on an object must be prepared by the compiler. If a base class implements a functions and it is not overridden by a derived class, the derived class must use the base class&#39;s implementation. But, if a derived class overrides, then it is that implementation that must be used.</p>
<p>
The mechanism that compilers use to achieve this is not mandated by the C++ standard, but most implementations use the concept of the &quot;virtual function table&quot;. For every class that has virtual members, a table is created which maps function names to their implementations.
If a class has virtual functions, the compiler creates a unique vtbl for the class. This vtbl has pointers to the implementations of the member functions.
Each instance holds a pointer (vptr) to its class's vtbl.
In the case of virtual inheritance, the derived class has a vtbl, which has a mix of pointers to member functions: those overridden in the derived class and those inherited.
</p>
<p>

{% codeblock Virtusl Member Functions - virtual_member_functions.cpp %}
class E {
  virtual void f() {}
};

E e;
dump_instance("A class with a virtual member function:", e, 1); // We supply the number of virtual methods
{% endcodeblock %}

GCC:<br />
<pre>
A class with a virtual member function:
  bfd3dfd4 -> instance (4 bytes):
    f8 aa 04 08 
    vtbl (0804aaf8 - 0804aafb) 1 entry:
      0804aaf8 -> fptr 0:
        08048cec 
        08048cec -> implementation (first 16 bytes):
          55 89 e5 5d c3 90 55 89 e5 8b 45 08 c7 00 e8 aa 
</pre>
Visual C++:<br />
<pre>
A class with a virtual member function:
  0012ff60 -> instance (4 bytes):
    e4 50 41 00 
    vtbl (004150e4 - 004150e7) 1 entry:
      004150e4 -> fptr 0:
        00405370 
        00405370 -> implementation (first 16 bytes):
          c3 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 
</pre>
</p>
<p>
This class's instances occupy 4 bytes, which hold the pointer to the class's virtual function table.
</p>

{% codeblock Virtual Member Functions - virtual_member_functions.cpp %}
class F {
public:
  F() : fdata(0xffffffff) {}; // Initialize with easy-to-spot data to make dumps clearer
  virtual int f(void) = 0;
private:
  int fdata;
};

class F1 : F {
public:
  F1() : f1data(0xf1f1f1f1) {}; // Initialize with easy-to-spot data to make dumps clearer
  virtual int f(void) { return 0x33333333; }
private:
  int f1data;
};

F1 f1;
dump_instance("A subclass of an abstract class:", f1, 1);
{% endcodeblock %}

GCC:<br />
<pre>
A subclass of an abstract class:
  bfd3dfb8 -> instance (12 bytes):
    d8 aa 04 08 ff ff ff ff f1 f1 f1 f1 
    vtbl (0804aad8 - 0804aadb) 1 entry:
      0804aad8 -> fptr 0:
        08048d30 
        08048d30 -> implementation (first 16 bytes):
          55 89 e5 b8 33 33 33 33 5d c3 55 89 e5 8b 45 08 
</pre>
Visual C++:<br />
<pre>
A subclass of an abstract class:
  0012ff78 -> instance (12 bytes):
    e0 50 41 00 ff ff ff ff f1 f1 f1 f1 
    vtbl (004150e0 - 004150e3) 1 entry:
      004150e0 -> fptr 0:
        00401140 
        00401140 -> implementation (first 16 bytes):
          b8 33 33 33 33 c3 90 90 90 90 90 90 90 90 90 90 
</pre>
</p>
<p>
Both compilers place the vtbl at the start of the instance memory, so a pointer to the instance is a pointer to a pointer to the vtbl. The instance data follows, laid out in "class derivation order".
</p>
<h1>Object dumping code</h1>
<p>
	The examples in this article are <a href="http://github.com/joeyates/c-plus-plus-study/tree/master/01_binary_layout/">available from Github</a>.</p>

The function `dump_instance()` prints the memory occupied by the object.

{% codeblock Dump Object Internals - dump_instance.cpp %}
template<class T> 
void dump_instance(const char * title, const T& t, int virtuals = 0) {
  cout << title << endl;
  stringstream sstr;
  sstr << "instance (" << object_size(t) << "):";
  dump_memory((unsigned char *) &t, sizeof(t), sstr.str().c_str(), 1);
  if (virtuals > 0)
    dump_vtable(t, virtuals);
  cout << endl;
}
{% endcodeblock %}

The rest of the code is handles printing blocks of memory, and the virtual function table:

{% codeblock Helpers - dump_helpers.cpp %}
// output formatting helper
void indent(int depth) {
  for(int i = 0; i < depth; ++i)
    printf("  ");
}

template <class T>
void dump_memory(const T * p, size_t items, const char * title, int depth) {
  indent(depth);
  printf("%08x -> %s\n", (unsigned int) p, title);
  // Make a format string that prints hex with correct number of leading zeroes
  stringstream sstr;
  sstr << "%0" << sizeof(T) * 2 << "x ";
  string format = sstr.str();
  indent(depth + 1);
  const T * end = p + items;
  for(const T * chunk = p; chunk < end; ++chunk) {
    printf(format.c_str(), *chunk);
  }
  printf("\n");
}

// Pretend all pointers are to int to reduce number of casts
template<class T> 
void dump_vtable(const T& t, int virtuals) {
  // In memory, the first bytes of the instance are occupied by the vtable
  int * vtbl = (int *) &t;
  int vtable_size = virtuals * sizeof(int);
  indent(2);
  printf("vtbl (%08x - %08x)", *vtbl, *vtbl + vtable_size - 1);
  printf(" %d entr%s:\n", virtuals, ((virtuals == 1) ? "y" : "ies"));
  for(int i = 0; i < virtuals; ++i) {
    int * fptr = ((int **) vtbl)[i];
    stringstream sstr;
    sstr << "fptr " << i << ":";
    dump_memory(fptr, 1, sstr.str().c_str(), 3);
    dump_memory((unsigned char *) *fptr, 16, "implementation (first 16 bytes):", 4);
  }
}

template<class T>
string object_size(const T& t) {
  stringstream sstr;
  sstr << sizeof(t) << " byte";
  if (sizeof(t) != 1)
    sstr << "s";
  return sstr.str();
}
{% endcodeblock %}

# Notes

The text assumes a 32-bit, little-endian  machine.

References:
<ul>
<li>http://www2.research.att.com/~bs/bs_faq2.html#layout-obj</li>
<li>http://www2.research.att.com/~bs/bs_faq2.html#sizeof-empty</li>
<li>http://www.cprogramming.com/tutorial/size_of_class_object.html</li>
</ul>
