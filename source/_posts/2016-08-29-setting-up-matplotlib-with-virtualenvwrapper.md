---
layout: post
title: "setting up matplotlib with virtualenvwrapper"
permalink: setting-up-matplotlib-with-virtualenvwrapper
date: 2016-08-29 08:32:07
comments: true
description: "setting up matplotlib with virtualenvwrappe (OS X)"
keywords: ""
categories:

tags:

---

I've just started the [machine learning course with Udacity ](https://www.udacity.com/course/intro-to-machine-learning--ud120). Things are going well, but there was one set up stumbling block I ran into: setting up matplotlib with virtualenvwrapper. The typical work flow, where you enter a virtualenv and then `pip install matplotlib` doesn't work. When trying to import `matplotlib` we get the error:

```
RuntimeError: Python is not installed as a framework. The Mac OS X backend will not be able to function correctly if Python is not installed as a framework. See the Python documentation for more information on installing Python as a framework on Mac OS X. Please either reinstall Python as a framework, or try one of the other backends. If you are Working with Matplotlib in a virtual enviroment see 'Working with Matplotlib in Virtual environments' in the Matplotlib FAQ
```

Yes, there are other options such as Anaconda. And if that solution works for you, by all means use it. I was curious if there was a way to accomplish this with virtualenv. It turns out that Matplotlib was happy on my machine as a global install and your virtualenv can simply reference it. Again, there may be reasons why this won't work for you (perhaps you need multiple versions of the library on your machine) but if it does work, here's how you do it.

```
$ pip install matplotlib  # note, not inside virtual env here
$ workon myenv
(myenv) $ toggleglobalsitepackages
Enabled global site-packages  # make sure global site-packages are enabled
# if they were disabled, enter this command again.
(myenv) $ python
>>> from matplotlib import pyplot
>>> # it worked!
```
