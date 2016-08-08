---
layout: post
title: More with Rspec
date: '2016-03-30T21:20:06+00:00'
permalink: more-with-rspec
---
Rspec is a powerful framework that lets us test all kinds of different things in our rails application. The more I use it, the more I am seeing just how many different parts of the application tests need to effect. For example, session data. Today, I came across this error on a simple test.


    QuestionsController POST create success redirects to index
     Failure/Error: expect {post(:create,
     NoMethodError:
       undefined method `authenticate' for nil:NilClass
     # ./app/controllers/questions_controller.rb:46:in `create'
     # ./spec/controllers/questions_controller_spec.rb:65:in `block (4 levels) in <top (required)>'
     # ./spec/controllers/questions_controller_spec.rb:65:in `block (3 levels) in <top (required)>'


This method needs to know who the signed in user is, because it implicitly draws on the user id to create a `Question`. So how to we simulate this in a test?

###current_user

The method `current_user` pulls the `User` object for who ever is signed in. It looks to the `session` hash to determine this. If there's no user in the session hash, any call to `current_user` will return `nil`. Can look at the definition to `current_user` to backward engineer a valid user, and then load up session data appropriatly? In a smaller rails application I've worked on, `current_user` is simply defined in the `ApplicationController`. A simple search found no reference to `def current_user`. What gives? It has to be defined somewhere. A bit more research turned up that current_user is defined implicitly in the devise gem (a gem for authentication) and not in the applicaiton codebase at all.  I took a look inside and it was a bit tricky to work out what was going on. Too abstract. Let's try another approch.

###Ask Google!
Fortunatly, Google came through for me again with [this article][1]. If we include these lines of code, we can fake the signing in of a user:

    @request.env["devise.mapping"] = Devise.mappings[:admin]
    sign_in user

Almost, but not quite. I run the test with these two lines added and get another error:

    Failure/Error: sign_in user
         NoMethodError:
           undefined method `sign_in' for #<RSpec::ExampleGroups::QuestionsController::ATest:0x007ff383c6c5b8>

I'm spoiled on Python! Requiring and including stuff in Ruby is less straight forward. How do I do something as simple pull in the method `sign_in`? More research, turns up that I need to have `include Devise::TestHelpers` at the top of the test. So all together it would look something like this.

    RSpec.describe QuestionsController, :type => :controller do
      include Devise::TestHelpers

      before(:each) do
        FactoryGirl.create :global_role_admin
        FactoryGirl.create :global_role_instructor
        FactoryGirl.create :global_role_user
        user = FactoryGirl.create :confirmed_user
        FactoryGirl.create :exercise
        FactoryGirl.create :question
        @request.env["devise.mapping"] = Devise.mappings[:user]
        sign_in user
      end

      describe "POST create success" do
        it "redirects to index" do

          question_params = FactoryGirl.attributes_for(:question)
          expect {post(
              :create, 
              {:question => question_params},
              {:user_id => User.first.id}
            ) 
          }.to change(Question, :count).by(1)
          expect(response).to redirect_to(questions_path)
        end
      end

And it works!


  [1]: http://luisalima.github.io/blog/2013/01/09/how-i-test-part-iv/
