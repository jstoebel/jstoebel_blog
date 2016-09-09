---
layout: post
title: "Implementing Python's String Manipulation in Batavia"
permalink: implementing-pythons-string-manipulation-in-batavia
date: 2016-09-04 07:41:34
comments: true
description: "Implementing Python's String Manipulation in Batavia"
keywords: ""
categories:

tags:

---

This past week I have been making contributions to [Batavia](https://github.com/pybee/batavia) an JavaScript implementation of the Python Virtual Machine. The lowest hanging fruit has been to implement all of the operators for each builtin type (example: `str.__add__()`, `str.__mul__()`.) An interesting case came up for me when I started work on `str.__mod__()`. This type seemed like it would be simple enough (moduluo operation on a string and anything makes no sense. You would think to just throw a TypeError and be done with it) However, the `%` token is overloaded for strings to do string formatting. So for example:

```
>>> x="spam"
>>> x="I like %s"
>>> y = "spam"
>>> x % y
'I like spam'
```

The picture is further complicated by the fact that there are numerous [kinds of markers](https://docs.python.org/2/library/stdtypes.html#string-formatting-operations) specifying different types and various other characters like `+`, `-` and others can add additional complexity.

To be clear, Python3 [recommends](https://pyformat.info/) using `.format` for string formatting now, but continues to support overloading the mod operator.

Starting to dive into the problem, I quickly found myself in over my head. Fortunately, [Brython](https://www.brython.info/) which is a full implementation of Python in JavaScript has already taken this problem on. I did find one deficiency however:

```
# using Brython
>>> x = "three args: %s | %s | %s"
>>> x % False
"three args: False | False | False"
```
It shouldn't accept that. Since there are three markers and one argument provided, an exception should be thrown: `TypeError: not enough arguments for format string`.  My thinking was that if the method could first ensure that the number of markers matches the number of arguments we could then proceed with formatting. Otherwise throw the right exception right off the bat.

To do that, I used a big scary regex to capture all of the markers:

`/\x25(?:([1-9]\d*)\$|\(([^\)]+)\))?(\+)?(0|'[^$])?(-)?(\d+)?(?:\.(\d+))?([b-gijosuxX])/g`

Yikes! Even though I don't follow all of it, it fortunately works in my browser and in node. Unfortunately, unit tests are run using phantomjs, a headless webkit, and the regex does not work there. That puts us in a situation where tests produce one result, but when fired up in the browser, produce a different result.

Through a bit of trial an error I found that phantomjs doesn't like any regex involving the literal `+` operator. I filed a ticket with [phantomjs](https://github.com/ariya/phantomjs/issues/14525) and [batavia](https://github.com/pybee/batavia/issues/247) and am waiting to see if the community can provide guidance on what to do next.
