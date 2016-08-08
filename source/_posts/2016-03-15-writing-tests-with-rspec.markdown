---
layout: post
title: Writing tests with Rspec
date: '2016-03-15T01:28:12+00:00'
permalink: writing-tests-with-rspec
---
With our work under way of version 0 of the Q&A forum, I thought I would begin to tackle writing unit tests. CodeWorkout uses Rspec for their tests, which is a popular testing framework for Ruby. I am familiar with the rival testing framework MiniTest, so Rspec will be new territory for me. Here's what I learned so far:

##Factory Girl

Factory Girl is a great way to dynamically generate test data for tests. Here is a simple factory to make a question and a response:

    FactoryGirl.define do

      factory :question do
        title "a question"
        body "question body text"
        tags "for loops; conditionals"
        user
        exercise

      end
    end

    FactoryGirl.define do
      factory :response do
        text "response text"
        user
        question
      end
    end


Hopefully the language is clear enough. a `:question` factory creates a new Question object with the specified `title`, `body` and `tags`.  I also specified that the question should have a `user` and an `exercise`. These are associated tables and in the case of user it doesn't make sense for a question to not have a user. I don't want to bother with matching the question with  a valid `exercise` or `user` so I simply say "this question will have an exercise, just pick one. The `:response` factory works in much the same way.

If we want to use factories in our Rspec tests we would say something like the following at the top of our test.

    before(:each) do
      FactoryGirl.create :confirmed_user, {:email => "test@test.com"}
      FactoryGirl.create :question
    end

The block before(:each) tells the test suite that the following should be run before each test. Specifically, a user is created and a question is created. Any data created in a before(:each) block is rolled back after each test. This is important because we don't want data created in one test to be lying around after another test, potentially messing up results. We want each test to start from square one. If for some reason we needed to create data once and persist throughout all tests we would set it up inside before(:all)

##Testing questions#new  

To start, I want to test the simple controller action `questions#new`. If this action is working correctly two things will happen:

* It will assign an instance variable `@question` which is a new `Question` object.
* It will return a successful HTTP response (200)


        describe "GET new" do
          it "returns http success" do
            get :new
            expect(response).to have_http_status(:success)
            assigns(:question).should be_a_new(Question)
          end
        end

This test is for the get new action. This test does three things:

* it performs a get request on the resource.
* it expects a successful response.
* it makes sure that `@question` was set up and it is a new instance of the `Question` class.

Now we can run it by typing  `rake spec SPEC=spec/controllers/questions_controller_spec.rb`

Assuming everything went correctly, we should get something like

    Finished in 0.17649 seconds (files took 3.8 seconds to load)
    1 example, 0 failures

If we get an error message we'll know what to fix.

I would like to continue building tests for our version 0 of the Q&A forum. This is an opportunity to use test driven development, wherein we can specify what we expect our code to accomplish and then write code based on those specs.
