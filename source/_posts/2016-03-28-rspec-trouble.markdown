---
layout: post
title: Rspec trouble
date: '2016-03-28T22:32:15+00:00'
permalink: rspec-trouble
---
I've begun implementing tests using the Rspec framework. Code Workout's tests are still a work in progress (no shade!) and I'm learning how it all fits together. Here's a run down of some obstacles I encountered while attempting to write a few tests for the questions controller.

I want to create a spec for `questions#index` like so:


    describe "GET index" do
      before(:each) do
       get :index
      end
      it "returns http success" do
        expect(response).to have_http_status(:success)
      end

      it "pulls all questions" do
        # get :index
        expect(assigns(:questions)).to eq(Question.all)
      end

      it "renders the index view" do
        # get :index
        expect(response).to render_template("index")
      end
    end

Here are some of the problems I encountered, and how to fix them:

 1. cannot load such file -- rails_helper.

    For what ever reason, my tests have `require 'rails_helper'`. This should be changed to `require 'spec_helper'`

 2. uninitialized constant SseHelper (NameError)

    Peeking at the file sse_helper_spec.rb I see the problem:

        RSpec.describe SseHelper, :type => :helper do
          pending "add some examples to (or delete) #{__FILE__}"
        end
    As suggested I commented it out (just for now) and it doesn't complain any more.

 3. A whole bunch of other problems trying to run the test suite:

    It looks like the test suite is still being worked on and won't run successfully. I did a little bit of research and found that I can easily run just my own tests by typing `spec path/to/your/test` Let's just do this instead of trying to fix all of the other tests. 

 4. My own tests will need some data that can be set up and torn down for each test. My first attempt is this:

        FactoryGirl.create :user
        FactoryGirl.create :exercise
        FactoryGirl.create :question

    and I get this error: `Couldn't find GlobalRole with 'id'=3`. The problem is that a user expects to be assigned a GlobalRole and none have been created.

 5. Include the factory?

    Almost. When I try to call the factory `global_role_user` I get the error: `uninitialized constant GlobalRoleUser`. For some reason the factory doesn't know which Model to look for so we have to tell it. Change the factory definition to `factory :global_role_user, :class => 'GlobalRole' do` and it'll know exactly what model to look for.

 6. And success!

Here is my revised setup:

    before(:each) do
      FactoryGirl.create :global_role_admin
      FactoryGirl.create :global_role_instructor
      FactoryGirl.create :global_role_user
      FactoryGirl.create :exercise
      FactoryGirl.create :question
    end

The user is created implicitly when I create a question and because the right role is in place, it fits in perfectly. Now when I call this test it runs successfully. Now on to writing more!
