---
layout: post
title: "string formatting in Batavia"
permalink: string-formatting-in-batavia
date: 2016-12-22 20:06:35
comments: true
description: "string formatting in Batavia"
keywords: ""
categories:

tags:
 - Python
 - JavaScript
 - Batavia
---

For the last few weeks I have attempted to crack Python's [old style string formatting](https://docs.python.org/2/library/stdtypes.html#string-formatting). A rudimentary version already exists in Batavia, but numerous features and corner cases are not yet implemented. I wanted to try to [round things out a little bit more.](https://github.com/pybee/batavia/pull/391)

## plan 1: parse using a regex

The current implementation of this functionality uses a long regex to parse each specifier after a few hours of trying to build off of the existing regex and getting no further, I decided this would be an unwise approach. There is a great deal of complexity in this system that it makes my head hurt just to think about. Instead I'm going to have to rewrite everything to handle all of the complexity.

## plan 2: rewrite everything

### detecting possible specifiers
If we can't write a regex to parse each specifier, we can at least write a regex to detect _possible_ specifiers. The much simpler regex `/(%.+?)(\s|$)/` will give me each possible specifier in the format string. From there I can run them through a function for processing/validation.

By the way, I had initially written a generator using `function*` syntax to abstract this functionality. It turns out when I went to run tests that Webpack didn't take too kindly to this. I could have invested more time figuring out why and how to fix it, but instead I decided to reimplement as a simple while loop.

### Parsing/validating possible specifiers

From there, I need to parse the possible specifier. The bad news is that tokens in specifiers are not modular, meaning that the meaning/validity of each token is affected by what comes before and after both within the same specifier and later in the expression. I need to iterate over the entire specifier to figure out what's going on and then assuming nothing's wrong, go over it again to do the substitution. Specifically this means a `Specifier` object which will.

 1. Determine if any conversion flags will be used. Some flags override others, so logic needs to be in place to catch this.
 1. Logic to determine what stage of the parsing we are in. The character `#` is legal, but not if it comes after an integer for example. I implemented a function called `getNextStep` which tells me what stage of processing should happen with the next character seen.  
 2. Determine minimum field width and precision. Things are made even _more_ complex by the fact that instead of passing in integers you can also pass a `*` meaning that the value will be determined by the next argument passed. This means that...
 3. As a specifier is being parsed, I also have to pass in an array of all available unused arguments. When we find a place where an argument is needed grab one off the array and save it to the object. If not enough arguments are available throw an error in the appropriate way (and yes, the error depends on context.)
 4. Finally, once everything is parsed, circle back and have the `Specifier` object perform the substitution. Here there are numerous places where special handling is needed such as places where white space or leading/trailing 0s must be tacked on, positive/negative signs must be added and so much more. Its a dance of converting things between `Number` and `String` `split`ing and `join`ing, adding extra decimal places and on and on and on...


As of this writing (12/22/16), my current draft comes in over 500 lines of JavaScript though it still needs to be cleaned up, rotten code and comments removed, better comments added etc. I also have a skeleton for unit tests, though they aren't all passing yet (see below).

## TODO

 1. This iteration doesn't handle key word arguments. I still need to add this.
 2. I have not yet worked out how to handle a literal `%`. For example. This shouldn't be _too_ hard.

{% highlight python %}

>>> "%s %%s" % 's'

's %s'

{% endhighlight %}

 3. I have run into a very strange problem where when I have anything that returns a string like `3.000`, the string is reduced to `3.0`. Given that the user needs to be able to specify a precision in decimal numbers, this won't work. I tried firing up a regular script that I run through node and this issue doesn't emerge, but I _do_ see it when running the tests, leaving me to wonder if Webpack may be the culprit.
 4. Further work on tests to ensure we're covering everything.
 5. Clarify code with comments, delete rotten code and comments.
 6. Don't believe [this sign](https://twitter.com/PyBeeWare/status/812114406230216704)

## Update

I found the problem with truncating string representations of floats. After poking around in the webpack build a little more, it turns out this wasn't the culprit at all. There is a function in the `test/utils.py` called `cleanse_javascript`. Part of this function looks for any string representation of a float, converts it to a float then back to a string again. Goodbye extra decimals! To fix this problem I changed this...

{% highlight python %}

for match in JS_FLOAT_DECIMAL.findall(out):
    out = out.replace(match, str(float(match)))

{% endhighlight %}

...with this...
{% highlight python %}

for match in JS_FLOAT_DECIMAL.findall(out):
    out = out.replace(match, str(Decimal(match)))

{% endhighlight %}
