---
layout: default
date: 2013-12-08
title: "Python Built-In Functions"
tags: Python wip
---

**This post is work in progress. Last updated {{page.date}}.**

# Python Built-In Functions

In this post I will try and dig through some of the source code behind the
[built-in functions of Python
2.7](http://docs.python.org/2.7/library/functions.html)
My hope is that by going through some of the source code of Python I will get to
appreciate
technical aspects of Python better.

The source code of the built-in functions of Python 2.7 seems to
[be here](http://hg.python.org/cpython/file/tip/Python/bltinmodule.c).

Going through a small subset of functions listed
[here](http://docs.python.org/2.7/library/functions.html)
it becomes apparent that tracking down the source code of built-in functions is
not trivial.

For now, I will just focus on those built-in functions whose source code I can
locate
[here](http://hg.python.org/cpython/file/tip/Python/bltinmodule.c).

## [`basestring()`](http://docs.python.org/2.7/library/functions.html#basestring)

Finding the source code of this in Python 2.7 seems a bit tricky.
I will come back to this later.

## [`bin()`](http://hg.python.org/cpython/file/tip/Python/bltinmodule.c#l344)

As we can see, this one just calls `PyNumber_ToBase` with the base argument set
to `2`.
Unfortunately, I have not been able to find the source code for
`PyNumber_ToBase` yet.

## [`bool()`](http://docs.python.org/2.7/library/functions.html#bool)

Another one whose source code I cannot locate.

## [`__build_class__`](http://hg.python.org/cpython/file/tip/Python/bltinmodule.c#l49)

This one is not in the list of built-in Python functions but the source code for
this is in
`bltinmodule.c` to this got me curious.

A mention of this function is
[here](https://groups.google.com/forum/#!msg/comp.lang.python/vp_p5ncjQDU/UayJqs
YrOisJ).

Since `__build_class__` is not
[exported as a built-in
method](http://hg.python.org/cpython/file/tip/Python/bltinmodule.c#l2385),
we cannot call it directly.

## [`__import__`](http://hg.python.org/cpython/file/tip/Python/bltinmodule.c#l199)

As the docstring reads, `__import__` is meant for use by the Python interpreter.
Programmatic import of modules is encouraged to be done with
[`importlib.import_module()`](http://docs.python.org/2.7/library/importlib.html)
.
