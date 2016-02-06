---
layout: post
title: "Trying Flow"
date: 2016-02-04 18:04:55 +0100
comments: true
categories: 
---
I'm interested in the possibility of running type checks on Javascript code.

Proponents of type checking believe that bugs can be avoided by indicating the
intended type (i.e. String, Number) of values (variables, functions and
classes), opponents say it adds work but doesn't significantly reduce bugginess.

I read through the intro to [Typescript][typescript-home] and it certainly
seems like a lot of work.

An alternative is Facebook's [flow][flow-home], it allows gradual typing -
you add only as many type annotations as you want. Here, I'm giving it a spin
by creating [flow-hello][flow-hello] - a "Hello World!" Example.

[typescript-home]: http://www.typescriptlang.org/
[flow-home]: http://flowtype.org/
[flow-hello]: https://github.com/joeyates/flow-hello

# Installation

The type checking system is written in OCaml, and you need to install a binary
or [compile from source][building-flow].

[building-flow]: https://github.com/facebook/flow#building-flow

I opted to use `npm` to install the binary globally:

```sh
$ sudo npm install flow-bin --global
```

# Project setup

Configure flow:

```sh
$ flow init
```

# A Bug

Here's a Javascript with a bug:

<figure>
  <figcaption>File: src/01-buggy.js</figcaption>
```javascript
function logLength(x) {
  console.log(x.length);
}

logLength("Hello");
logLength(1);
```
</figure>

This code logs `undefined` when the number `1` is passed to the function
`logLength`, as `1` does not have a `length` attribute:

```sh
$ node src/01-buggy.js
5
undefined
```

As is, flow does not analyze the file:

```sh
$ flow check
Found 0 errors
```

# Activate flow

Flow is activated by adding `@flow` to the first comment in any file:

<figure>
  <figcaption>File: src/02-with-flow.js</figcaption>
```javascript
/* @flow */

function logLength(x) {
  console.log(x.length);
}

logLength("Hello");
logLength(1);
```
</figure>

```sh
$ flow check
src/02-with-flow.js:3
3:   console.log(x.length);
                   ^^^^^^ property 'length'. Property not found in
3:   console.log(x.length);
                 ^ Number

Found 1 error
```

That's good, as it explains how we get `undefined` in the output, but it's not
clear that this happens due to the second invocation of the function.

# Annotate function parameters

Now, let's indicate the intended type of the `x` parameter so that calls with
parameters of the wrong type will be pointed out.

<figure>
  <figcaption>File: src/03-parameter-annotations.js</figcaption>
```javascript
/* @flow */
function logLength(x: string) {
  console.log(x.length);
}

logLength("Hello");
logLength(1);
```
</figure>

As `flow check` checks all the `.js` files it finds, I'll run pass the file
contents to flow via stdin:

```sh
$ flow check-contents < src/03-parameter-annotations.js
7: logLength(1);
   ^^^^^^^^^^^^ function call
7: logLength(1);
             ^ number. This type is incompatible with
2: function logLength(x: string) {
                         ^^^^^^ string


Found 1 error
```

That's good - we know which call caused the problem.

# Stripping type annotations with Babel

Unfortunately, we can no longer simply run the code:

```javascript
$ node src/03-parameter-annotations.js
function logLength(x: string) {
                    ^
SyntaxError: Unexpected token :
...
```

We can use [Babel][babel] to [remove type annotations][transform-flow-strip-types]
when we want to run the code.

[babel]: http://babeljs.io/
[transform-flow-strip-types]: https://babeljs.io/docs/plugins/transform-flow-strip-types/

Setting up Babel requires a bit of work:

<figure>
  <figcaption>File: package.json</figcaption>
```json
{
  "scripts": {
    "babel": "babel ..."
  }
}
```
</figure>

```sh
$ npm install --save-dev babel-cli
...
$ npm install --save-dev babel-plugin-transform-flow-strip-types
...
```

<figure>
  <figcaption>File: .babelrc</figcaption>
```json
{
  "plugins": ["transform-flow-strip-types"]
}
```
</figure>

Now we can run babel:

```sh
$ node_modules/.bin/babel src/03-parameter-annotations.js

function logLength(x) {
  console.log(x.length);
}

logLength("Hello");
logLength(1);
```

We get normal Javascript as output.

# Conclusion

So, I got what I wanted: help with figuring out an error cuased by calling a
function with a parameter of the wrong type.

But, it's truly a "Hello World!", there is a whole type specification system
yet to learn and try out.

I like the idea of being able to annotate just as much of my code as I like, so
I think I'll be using Flow on my next Javascript project.
