---
layout: post
title: "Implementing mktime in Batavia or why CPython is a better choice for building a time machine"
permalink: implementing-mktime-in-batavia
date: 2016-10-16 14:35:43
comments: true
description: "Implementing mktime in Batavia"
keywords: ""
categories:

tags:
 - Python
 - JavaScript
 - Batavia

---

The Python function `mktime` tells us how many seconds a datetime is from the epoch. It is based on the C function of the same name and while this makes it run fast, it also means it is platform dependent. The goal of Batavia is to bring Python to the web browser meaning platform dependencies need to be ironed out.

## How early is too early?

The lower bound for `mktime` (the earliest possible date) varies widely across systems. On my Mac for example trying to evaluate with 1900-01-01 produces an `OverflowError`. This seemed like enough of a non-trivial example to build that restriction into Batavia. Imagine the frustration is a developer was working in Ubuntu and was burned when their OS X users got strange results. It didn't seem right to set the cutoff date equal to the OS X cutoff, so I just set the cutoff as 1900-01-01. In the future I suppose Batavia could identify the user's OS and make this determination from there, but for now this seemed like a good enough starting place.

## How late is too late?

Looking into the future, `mktime` can move _very_ far into the future before throwing an `OverflowError`. JavaScript however [has its own limits](http://ecma-international.org/ecma-262/5.1/#sec-15.9.1.1) on how far forward and back it may travel from 1970. Specifically, we can go no further into the future than than 275760-09-12. Since Batavia is written in JavaScript, there is simply no way around it, we can't handle dates later than that. Sorry folks trying to build a time machine with Batavia, use CPython instead!

## How to test this?

This brings us to an interesting problem that until now I had not encountered in Batavia. For most tests in Batavia we use `assertCodeExecution` which runs the Python bytecode through CPython and Batavia and asserts that the output is the same. This is normally fine, because most of CPython behaves the same regardless of platform. Not so much with `mktime` which is based on a C function that varies across platforms. Since `mktime` won't match the behavior of CPython across all platforms we can't use `assertCodeExecution`

Instead we use `assertJavaScriptExecution`. Here, we run some code through Batavia and assert the output is equal to a string we feed it. This almost gets the job done save for two exceptions:

The first is testing the lower bound for dates that wont throw an `OverflowError`. For this test, I am not interested in what the specific value is returned so much as it _doesn't_ throw an `OverflowError`. Yes, I could predict the time using `localtime` and `gmtime` (see below for that) but here I only really care what _isn't_. To that end I submitted a patch to `assertJavaScriptExecution` to optionally assert the output doesn't match the predicted:

The second challenge is how to test when the natural upper bound of JavaScript is reached. Specifically, we need test coverage for throwing the `OverflowError` in the snippet below:

```
var date = new Date(sequence[0], sequence[1] - 1, sequence[2], sequence[3], sequence[4], sequence[5], 0)

if (isNaN(date)){
    throw new batavia.builtins.OverflowError("signed integer is greater than maximum")
}

var seconds = date.getTime() / 1000;
return seconds.toFixed(1);
```  

The trouble here is that I don't yet know what timezone the machine running the tests is in and therefore can't determine the latest possible date that will pass. To determine the time zone offset (how many hours ahead/behind from GMT) we use `localtime().tm_hour - gmtime().tm_hour`. I can then use that number to determine what datetime to test with.
