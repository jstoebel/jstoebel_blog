---
layout: post
title: "Implementing struct_time in Batavia"
permalink: implementing-struct_time-in-batavia
date: 2016-09-28 08:42:32
comments: true
description: "Implementing struct_time in Batavia"
keywords: ""
categories:

tags:
 - Python
 - JavaScript
 - Batavia
---

Recently I have been contributing to [Batavia](https://github.com/pybee/batavia) a JavaScript implementation of the Python Virtual Machine. The project is still in its infancy, so lots of core functionality still needs to be added. To that end I am trying my hand at implementing [`time.struct_time`](https://github.com/jstoebel/batavia/blob/struct_time/batavia/modules/time.js#L27)

The `struct_time` constructor accepts as its argument any sequence. The sequences `list`, `str`, and `tuple` are the simplest cases. We can slice and iterate over them as we would expect. The types `bytes`, `bytearray`, `frozenset`, `set` and `range` don't behave as easily. But no problem, we just convert to a tuple and we're in business.

That leaves us with `dict` and `bytearray`. If `struct_time` is passed a `dict` type the values need to be determined by examining the keys. Presently, we have not implemented `dict.keys()` and therefore can't handle the `dict` type. Similarly, `bytearray` does not yet have `__iter__` implemented. We need a way to iterate over the sequence passed.

Finally, the method `__reduce__` is not yet implemented for `struct_time`. Reduce tells an object how it should respond to pickling, and I'm not sure if batavia will support picking.
