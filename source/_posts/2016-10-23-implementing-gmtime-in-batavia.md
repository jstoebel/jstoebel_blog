---
layout: post
title: "Implementing gmtime in Batavia"
permalink: implementing-gmtime-in-batavia
date: 2016-10-23 19:11:24
comments: true
description: "Implementing gmtime in Batavia"
keywords: ""
categories:

tags:
 - Python
 - JavaScript
 - Batavia

---

Following up from my [last post]({{ base }}/implementing-mktime-in-batavia/), I worked today on [implementing `gmtime`](https://github.com/pybee/batavia/pull/336) in the `time` module. This function is essentially the opposite of `mktime` by converting number of seconds since the epoch to a `struct_time` object. This was a relatively straightforward task, though like `mktime` I ran into some snags.

### range of inputs

Like `mktime` there is a limit to how far into the future or past you may go and that limitation is platform specific since this function is a wrapper around a C function. We're also limited by the [ECMA spec](http://ecma-international.org/ecma-262/5.1/#sec-15.9.1.1), restricting the range of inputs for `Date` in JavaScript. On my Mac, the restriction for CPython is _much_ further into the future than the ECMA limitation, making JavaScript the limiting factor here. It remains to be seen if this is the case for other platforms, so we may need to consider more sophisticated logic in the future, perhaps involving inspecting the user's operating system to determine the behavior of these functions.

### tests

`gmtime` may be called with zero arguments in which case it returns a `struct_time` representing the current time. Attempting to test this with `assertCodeExecution` presents a problem. Normally we hand some python code to `assertCodeExecution` which runs it through both CPython and Batavia and ensures the output is the same. The problem here is that there is a delay of a few seconds between running the code on the two VMs resulting in a failed test. To address this I had to [reach a bit deeper into the testing utilities](https://github.com/pybee/batavia/pull/336/files#diff-f923a262959de00a7acd87b0c90acfcfR427). I ran the code through both `runAsJavaScript` and `runAsPython`, parsed the results using a regex and then converted the result to seconds since epoch using `mktime`. If the results are within 5 seconds of each other I am saying that they pass, giving even a sluggish testing environment some leeway.
