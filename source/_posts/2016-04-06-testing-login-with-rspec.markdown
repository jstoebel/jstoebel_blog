---
layout: post
title: Testing login with Rspec
date: '2016-04-06T01:35:57+00:00'
permalink: testing-login-with-rspec
---
I may have spoken too soon in my last blog post about logging in a user inside of Rspec. It turns out that while I was able to work out how to login a user inside of a spec file like so:

    before(:each) do
      admin = FactoryGirl.create :global_role_admin
      inst = FactoryGirl.create :global_role_instructor
      user = FactoryGirl.create :global_role_user

      user = FactoryGirl.create(:admin,
        email:      "admin@test.org")
      FactoryGirl.create(:instructor_user,
        email:      "inst_user@test.org")
      FactoryGirl.create(:confirmed_user,
        email:      "reg_user@test.org")

      FactoryGirl.create :exercise
      FactoryGirl.create :question
      @request.env["devise.mapping"] = Devise.mappings[:user]
      sign_in user
    end


What I would like to do is have an external module defined once and call it wherever I need to have a test pretend to login a user like so:

    module TestHelper
      require 'spec_helper'


      ALL_USERS = ["administrator", "instructor", "regular_user"]

      def self.login_as(user_type, req)
        #user_type: string repsenting type of user to login. Can be, :administrator, :instructor, or :regular_user
        user = User.find_by(global_role_id: GlobalRole.send(user_type))
        @request.env["devise.mapping"] = Devise.mappings[:user]
        sign_in user
      end
    end

The problem is that I simply don't have access to `@request` inside of my module.  If its called inside of the spec, it works. If its not, it doesn't.  Ruby modules are a bit more complicated than in Python (needlessly I might argue). While I could set up this functionality inside of each spec, that would not be DRY. This may require a different approach...

##Ask Stackoverflow

Fortunately, the good community at Stackoverflow came through for me and pointed me to this [resource] [1]. My need is somewhat different here. I would like to loop over all valid roles for a particular controller action and then ensure that the expected outcomes occurs. I don't want to simply log in a user each time, I want to write a test once and then try it out across several users.

##A different approach

After some sleep, I am reflecting on the fact that when you are working in a framework, you don't always get to solve a problem the way you might first think to approach it. Sometimes there is a better, clearer path if you can just be open to following it. or to quote the Zen of Python "There should be one-- and preferably only one --obvious way to do it." I think this might be an instance of me struggling to do it _my_ way, when in fact the framework might be pushing me to do it a different way. So with that in mind, here is another idea:

    describe "a test" do

      allowed_users.each do |n|
        context "for n" do
          @request.env["devise.mapping"] = Devise.mappings[:user]
          sign_in user

          #run the test
        end
      end
    end

I would still much prefer to not have to type out `@request.env["devise.mapping"] = Devise.mappings[:user]` each time and instead user a helper method, but this might have to be saved for a refactor later on.

 
  [1]: https://github.com/plataformatec/devise/wiki/How-To:-Test-controllers-with-Rails-3-and-4-(and-RSpec)
