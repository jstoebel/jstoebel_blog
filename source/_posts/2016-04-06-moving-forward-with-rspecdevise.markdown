---
layout: post
title: Moving forward with Rspec/Devise
date: '2016-04-06T23:33:00+00:00'
permalink: moving-forward-with-rspecdevise
---
After many hours of struggling mightily, I've come up with fairly (though not completely) DRY solution for running one test as multiple users.

##The Problem

I want to write a module that will let me simulate the login of a type of user, and then call that method in various tests. It would be very unDRY to have to define this as a private method in each test. I followed [this wiki][1] for help on this. 

    module ControllerMacros
      def login_admin
        before(:each) do
          @request.env["devise.mapping"] = Devise.mappings[:admin]
          sign_in FactoryGirl.create(:admin) # Using factory girl as an example
        end
      end

      def login_user
        before(:each) do
          @request.env["devise.mapping"] = Devise.mappings[:user]
          user = FactoryGirl.create(:user)
          user.confirm! # or set a confirmed_at inside the factory. Only necessary if you are using the "confirmable" module
          sign_in user
        end
      end
    end

At first glance I couldn't understand how this would fit my needs. What I needed was something like:

    start a test
        iterate over each allowed user
            login user
            run the test

It seemed to me that I needed the ability to login a user of my choice, run a test and then repeat. This business with a before block worked for logging in one specific user, but needing to login a user based on a passed in argument didn't seem possible in this model.

After some more struggle and thinking,  I came to a revelation: rspec supports loops! It's actually perfectly fine to do this:

    describe "a test" do
        3.times do |n|
            context n.to_s do
                #the test here
            end
          end
        end
      end


My need for multi user testing my actually be possible. I wrote this module:

    module ControllerMacros

      ALL_ROLES = ["administrator", "instructor", "regular_user"]
      def login_as(role_name)
        before(:each) do
          #grab the right user
          @request.env["devise.mapping"] = Devise.mappings[:admin]
          sign_in user
        end
      end
    end

The question then is, how can I user the method's argument to pull the right user. If I look at the model for `GlobalRole` we see three useful class methods:


    def self.administrator
      find(ADMINISTRATOR_ID)
    end

    def self.instructor
      find(INSTRUCTOR_ID)
    end

    def self.regular_user
      find(REGULAR_USER_ID)
    end

I can call these methods to get the desired role, and then find a user with the corresponding role. But how do I make the right method call based on string argument? Enter the `send` class method. I can call `send` with a string on any class and get back the corresponding method.  So to put it all together my method would be:

    def login_as(role_name)
      before(:each) do
        user = User.find_by(:global_role_id => (GlobalRole.send(role_name)).id)
        sign_in user
      end
    end

From there, I just need to add factories in that method to create the right role and user right? Wrong. I was getting strange results when trying to create an instructor and getting back a user with global_role_id of 1. What gives? It turns out that the factories, the way they are constructed now, assumes that roles will be constructed in this order (admin, instructor, regular_user). If we try to create the instructor role first, it will have global_role_id = 1 which in the model is tied to admin. We might want to address this for later. The work around is to have to define all roles and users for each test.

At any rate, the final, DRY(ish) test would look like this.

    #controller_macros.rb
    module ControllerMacros

      ALL_ROLES = ["administrator", "instructor", "regular_user"]
      def login_as(role_name)
        before(:each) do
          
          user = User.find_by(:global_role_id => (GlobalRole.send(role_name)).id)
          sign_in user
        end
      end
    end

    #questions_controller_spec.rb
    require 'spec_helper'

    RSpec.describe QuestionsController, :type => :controller do
      include Devise::TestHelpers

      before(:each) do
        FactoryGirl.create :global_role_admin
        FactoryGirl.create :admin, {:email => "admin@test.org"}
        FactoryGirl.create :global_role_instructor
        FactoryGirl.create :instructor_user, {:email => "instructor@test.org"}
        FactoryGirl.create :global_role_user
        FactoryGirl.create :confirmed_user, {:email => "student@test.org"}
        FactoryGirl.create :exercise
        FactoryGirl.create :question
      end


      describe "GET index" do
        ControllerMacros::ALL_ROLES.each do |r|
          context "as #{r}" do
            login_as r
            it "returns http success" do
              get :index
              expect(response).to have_http_status(:success)
            end

            it "pulls all questions" do
              get :index
              assigns(:questions).should
              expect(assigns(:questions)).to eq(Question.all)
            end

            it "renders the index view" do
              get :index
              expect(response).to render_template("index")
            end
          end
        end
      end

  [1]: https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-3-and-4-(and-RSpec)
