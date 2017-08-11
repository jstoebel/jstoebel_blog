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

My [nightlife app](https://github.com/jstoebel/nightlife) app makes use of a few external services in production. This presents some tricky problems when we are trying to test the code. Do I really want my test suite to have to hit the Yelp api during tests? Common advice is that we want to create a dependable, consistent testing environment. This means that we shouldn't be hitting  external services during tests. So what do we do. Enter mocks! I wrote a bit about mocks in [earlier post](/a-no-frills-jump-into-fakes-mocks-and-stubs) but I wanted to take a minute to dive in a little deeper with some examples in `sinon` and `mockery`:

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

What happens though when the thing we want to mock is `import`ed into another module. We can't change production code to suite our tests, so what do we do? The solution I reached for was [Mockery](https://github.com/mfncooper/mockery). Mockery lets you change what happens when you `require` a module. Rather than pulling in the module in question it instead will pull in your mock object. Here's an example from my project. I have this function that makes a request to the yelp api:

{% highlight javascript %}

const fetchBars = (token, location) => {
  // token (str): the yelp access token
  // return: a promise containing bars in the area

  const yelp = require('yelp-fusion');
  return yelp.client(token)
  .search({
    location: location,
    categories: 'bars',
  });
};

{% endhighlight %}

And here is my test code:

{% highlight javascript %}
    
    // the yelp api returns a promise containing your data. we're setting up 
    // our own promise containing something similar.
    const resultsSuccess = new Promise((resolve, reject) => {
      resolve({
        jsonBody: {
          businesses: sampleData,
        },
      });
    });
    it('succeeds on first try', (done) => {

      // this will be the mock object that replaces the yelp api
      const yelpMock = {
        client: function() {
          return {
              search: sinon.stub().returns(resultsSuccess),
          };
        },
      };

      // here's where the magic happens. This will cause all _future_ calls to 
      // `require('yelp-fusion')` to pull in our mock. They key there is 
      // future calls. Make sure you register the mock before the `require`ing 
      // happens

      mockery.registerMock('yelp-fusion', yelpMock);

      request(app)
        .get('/api/bars/search/12345')
        .expect(200)
        .then((resp) => {
          expect(JSON.stringify(resp.body.bars))
            .to
            .equal(JSON.stringify(sampleData));
          done();
        });
    }); // end test

{% endhighlight %}

Great! The one confusing part I had with this solution was that I had to `require` the yelp api inside my function. The more common approach with ES6 is to use `import` at the top of the module. The problem with that is that The controller file is executed when my `routes` file pulls it into my express router all of which happens before I'm able to run my test code. If anyone has a better solution than function level `require` I would love to hear it but for now this is the best solution I've got.