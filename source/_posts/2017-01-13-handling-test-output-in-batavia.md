---
layout: post
title: "Cleanse Your Batavia Test Output Using This One Weird Trick"
permalink: handling-test-output-in-batavia
date: 2017-01-13 11:16:23
comments: true
description: "Cleanse Your Batavia Test Output Using This One Weird Trick"
keywords: ""
categories:

tags:
 - Python
 - Javascript
 - Batavia
---

Batavia has a great framework for running tests. Essentially you write Python that results in some kind of output. The test runs that code through both C Python and Batavia, then you can compare those outputs. In most cases, we want to know that the outputs match. The author of the test doesn't need to worry about hard coding any kind of expectation for what the output should be. This gets tricky however in that the output from C Python and Batavia have to be cleaned up a little bit to ensure they match. For example, the output `3.000` is transformed to `3.0`. In the case where I print a float, this is the correct output. But what happens if I hand it ` "%.3f" % 3`. In this case `3.000` is the correct output. The extra zeros shouldn't be dropped. The problem here is that output is naive; nothing about the output tells us what produced it.  I want to have sensible defaults for output transformation but also be able to pass in options to skip certain transformations. This way I can avoid having to refactor existing tests, but have more flexibility in new tests when I need them.

The existing transform functions already perform several transforms on output. Asking a writer of a test pass directly into the assertion which parts to skip could get messy. Instead I wrapped it up state into two new classes, `JSCleaner` and `PYCleaner`.

{% highlight python %}
class JSCleaner:

    def __init__(self, err_msg=True, memory_ref=True, js_bool=True, decimal=True, float_exp=True, complex_num=True,
        high_percision_float=True, test_ref=True, custom=True):

        self.transforms = {k:v for k, v in locals().items() if k != 'self'}

    def cleanse(self, js_input, substitutions):
        """
        cleanse output from javascript
        """
        ...

class PYCleaner:

    def __init__(self, err_msg=True, memory_ref=True, float_exp=True, complex_num=True,
        high_percision_float=True, test_ref=True, custom=True):

        self.transforms = {k:v for k, v in locals().items() if k != 'self'}

    def cleanse(self, js_input, substitutions):
        """
        cleanse output from javascript
        """
        ...

{% endhighlight %}

This way, all transforms are performed be default. Then inside assertions that need to perform those transformations we change them to call the `cleanse` method instead of calling `cleanse_javascript` and `cleanse_python`.

I also created a decorator providing a simple interface for injecting a `JSCleaner` and `PYCleaner` object into tests that have needs other than the default.

{% highlight python %}

  def transforms(**transform_args):
      """
      injects a JSCleaner and PYCleaner object into the function
      use this as a decarator to configure which transformations should be performed
      """
      def real_decorator(function):

          def wrapper(self, *args, **kwargs):

              # js_cleaner
              js_excludes = ['py_test_script', 'py_str_excep']
              js_params = {jsk : jsv for jsk, jsv in transform_args.items() if jsk not in js_excludes}
              js_cleaner = JSCleaner(**js_params)

              # py_cleaner
              py_excludes = ['js_bool', 'decimal', 'float_exp']
              py_params = {pyk : pyv for pyk, pyv in transform_args.items() if pyk not in py_excludes}
              py_cleaner = PYCleaner(**py_params)

              res = function(self, js_cleaner, py_cleaner, *args, **kwargs)
              return res
          wrapper.__name__ = function.__name__
          wrapper.__doc__ = function.__doc__
          return wrapper

      return real_decorator

{% endhighlight %}

Then if you have a test that needs to skip any transformations:

{% highlight python %}

  @transforms(
      js_bool=False,
      decimal=False,
      float_exp=False,
      memory_ref=False
  )
  def test_basic(self, js_cleaner, py_cleaner):
      tests = adjust("""
      print(">>> 'Hello, world!'")
      print('Hello, world!')
      """)

      # the cleaner objects are passed into the assertion.
      self.assertCodeExecution(tests, js_cleaner=js_cleaner, py_cleaner=py_cleaner)

{% endhighlight %}
