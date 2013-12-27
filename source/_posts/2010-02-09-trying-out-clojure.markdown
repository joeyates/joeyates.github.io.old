--- 
layout: post
typo_id: 8
title: Trying out Clojure
---
<h3>Dependencies and Java configuration</h3>
<h4>
	1. Java</h4>
<p>
	Check you have Sun Java installed:</p>
<pre>$ java -version
java version "1.6.0_16"
Java(TM) SE Runtime Environment (build 1.6.0_16-b01)
Java HotSpot(TM) Server VM (build 14.2-b01, mixed mode)
</pre>
<p>
	If you see 'OpenJDK', you are using GNU Java: GCJ. It's better to switch to Sun Java.</p>
<h4>
	2. libjline</h4>
<p>
	On Ubuntu:</p>
<pre>$ sudo apt-get install libjline-java libjline-java-doc</pre>
<h3>
	Clojure</h3>
<p>
	Clojure is under quite rapid development still, so it's better to clone the git repository and compile it:</p>
<pre>$ cd /usr/share
$ sudo git clone git://github.com/richhickey/clojure.git
$ cd /usr/share/clojure</pre>
<p>
	The 'ant' package is needed for compilation.</p>
<pre>$ sudo apt-get --no-install-recommends install ant ant-optional
</pre>
<p>
	(The parameter '--no-install-recommends' avoids the installation di GCJ.)</p>
<pre>$ sudo ant
$ cd /usr/share/java
$ sudo ln -s /usr/share/clojure/clojure.jar</pre>
<h4>
	Check it works</h4>
<pre>$ java -cp /usr/share/java/clojure.jar clojure.main -e '(str "Hello World!")'
"Hello World!"
</pre>
<h3>
	REPL</h3>
<p>
	The best way to start using Clojure is via the REPL:</p>
<pre>$ java -cp /usr/share/java/clojure.jar:/usr/share/java/jline.jar jline.ConsoleRunner clojure.main
user=> (defn say-hello [] (str "Hello World!"))
#'user/say-hello
user=> (say-hello)
"Hello world!"
user=> [Ctrl+D]
</pre>

## Startup script

The following script was adapted from [here](http://paulbarry.com/articles/2007/12/22/getting-started-with-clojure)

Create it as 'clj' in a directory in your PATH:

    #!/bin/bash
    CLOJURE=/usr/share/java/clojure.jar
    JLINE=/usr/share/java/jline.jar
    for i in $CLOJURE $JLINE do
      if [ ! -e "$i"  ]; then
        echo "File not found: $i"
        exit 1
      fi
    done
    if [ -z "$1" ]; then
      java -cp $CLOJURE:$JLINE jline.ConsoleRunner clojure.main
    else
      CLASS=$1
      shift
      java -cp $CLOJURE clojure.lang.Script $CLASS -- $*
    fi

Remember to make it executable.
