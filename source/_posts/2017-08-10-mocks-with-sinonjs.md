---
layout: post
title: "Mocks with Sinon.js"
permalink: mocks-with-sinonjs
date: 2017-08-10 16:09:45
comments: true
description: "Mocks with Sinon.js"
keywords: ""
categories:

tags:

---

My [nightlife app](https://github.com/jstoebel/nightlife) app makes use of a few external services in production. This presents some tricky problems when we are trying to test the code. Do I really want my test suite to have to hit the Yelp api during tests? What about the database? Common advice is that we want to create a dependable, consistent testing environment. This means that we shouldn't be hitting either the database or external services during tests. So what do we do. Enter mocks! I wrote a bit about mocks in [earlier post](/a-no-frills-jump-into-fakes-mocks-and-stubs) but I wanted to take a minute to dive in a little deeper with some examples in `sinon` and `mockery`:

## Mocking an external service

Sinon makes creating a mock straight forward enough. We create a mock object to pretend to be a real object. We can dictate its behavior and make assertions on how it was called. This is straight from the documentation. 

{% highlight javascript %}
"test should call all subscribers when exceptions": function () {
    var myAPI = { method: function () {} };

    var spy = sinon.spy();
    var mock = sinon.mock(myAPI);
    mock.expects("method").once().throws();

    PubSub.subscribe("message", myAPI.method);
    PubSub.subscribe("message", spy);
    PubSub.publishSync("message", undefined);

    mock.verify();
    assert(spy.calledOnce);
}

{% endhighlight %}

What happens though when the thing we want to mock is `import`ed into another module. We can't change production code to suite our tests, so what do we do?