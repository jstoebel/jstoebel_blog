---
layout: post
title: "A No Frills Jump Into Fakes, Mocks and Stubs"
permalink: a-no-frills-jump-into-fakes-mocks-and-stubs
date: 2017-04-05 10:27:12
comments: true
description: "A No Frills Jump Into Fakes, Mocks and Stubs"
keywords: ""
categories:

tags:

---

As a semi self taught programmer, I've always felt an imposter syndrome flare up when discussion of mocks, stubs, fakes, etc comes up. I knew generally that they were used for testing things that should not or can not be used in a testing environment, but I also knew that you can't really understand and feel comfortable with something until you use it. No more. For anyone else who's felt intimidated by this concept, let me try to dispel that fear by giving you the basic run down:

## Doubles: The super class

When we're testing our code, there are certain resources that we can't or don't want use. So instead we create other objects to stand in for their behavior. The general term for these objects are doubles. I'll explain a few of those types here.

### Stubs: Let's not and say we did
Let's say we have an app that calls out to an API to handle payment. It would be both expensive and silly to have a dedicated credit card that we charge each time we want to ensure our app is running properly. Nor do we need to. Assuming that API is maintained by a reputable company (and if its not why are we using it), its not our job to worry about it failing (at least in the test suite anyway). Even if it does fail (any API can) there would be nothing we can do about it. Its outside the boundary of our responsibility and _therefore we don't need to test it_. Instead we will create a stub which is simply a stand in object that gives a _canned response_ every time you use it. Let's say in this very simple payment API you send credit card details (number, expiration, name, amount to charge, etc) and get back either `true` meaning "yes this card was accepted" or `false` meaning "no this card was rejected". Since this payment API is not our responsibly we can just stub it out. This means that when running tests we say "here I want to test how my code responds when payment is accepted. Rather than hitting the payment API, let's just pretend I did and the response was `true`".

### Mocks: Are you using me the way you should?

The word mock is sometimes used to refer to the general idea of a test object standing in for a production ready object. Here I am referring to something more specific. _A mock is an object that has certain expectations on the way it is called_. Let's say you have a production object that makes a call over the network, and you want to implement a test that ensures this network call not only functions properly but is made exactly once. A mock object would be designed to handle this network call and assert that it is made exactly once. If a network call is made twice or not at all, the test will fail, even if everything else works exactly as expected.

### Fakes: As real as you need them to be

Sometimes though we need to test behavior that is inside our realm of  responsibility but for certain reasons we want to avoid doing the same thing as we would in production. Maybe that real object has methods that involve reading from or writing to disk, which when compounded can really slow your test suite down. I encountered this exact problem recently on a project. I was using the [Ruby Gem Roo](https://rubygems.org/gems/roo/versions/2.7.1) to parse a spreadsheet. Roo takes an open file and returns an object with a bunch of methods to read from or write to that file. Rather than having my test suite write some data to a spreadsheet, save to disk, then open that file again for processing (which would inflict a performance hit) I created a fake. This fake object has some data and methods bound to it _that are similar enough to the real thing_. How similar? Similar enough to trick my app into thinking its the real thing.

Roo has a real class called `Sheet` which represents a spreadsheet. Let's say my app needs to call the methods `row` and `cell` on instances of this class. `row` accepts an integer and returns an array representation of the corresponding row. `cell` is handed a column and row coordinate and responds with the corresponding value. No problem, I'll just make a small fake that implements those features.

{% highlight ruby %}
    class Sheet
      # a fake of a Roo sheet object

      def initialize(arr)
        # arr: a 2D array, similar to the structure of data in a spreadsheet
        @arr = arr
      end

      def row(idx)
        # index of row to find (starts at 1)
        return @arr[idx-1]
      end

      def cell(row, col)
        # cell to find (both values starts at 1)
        return @arr[row-1][col-1]
      end

    end

{% endhighlight %}

And that's it! Because Ruby is weakly typed, we simply hand the app this fake object. My app will call `row` and `cell` on that object just like it would on the real thing. The fake object behaves the same way with respect to these two methods. The caller doesn't know (or care) about the difference. It won't notice that all of the other methods aren't implemented because it won't try to call them. Are there corner cases to these real methods that I haven't covered in my implementation? Without a doubt. But they won't come up in my use case. If later on they do I will add them. That's what a fake is: as real as it needs to be and no more.
