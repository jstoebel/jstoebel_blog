---
layout: post
title: DRYing up Rspec tests with let and subject
date: '2016-04-12T21:23:51+00:00'
permalink: drying-up-rspec-tests-with-let-and-subject
---
The Ruby/Rails community is a bit obsessed with DRY (Don't Repeat Yourself). And for good reason! Less typing means fewer chances of error and easier maintainability. This is especially evident in the Rspec, one of the leading testing frameworks for rails.

Let's take a look an an actual spec that I've been working on for the questions controller:

    describe "POST destroy success" do
      #should be able to destroy when the post belongs to you
      #or if you are an admin
      ControllerMacros::ALL_ROLES.each do |r|
        context "as #{r}" do
          login_as r

          it "redirects to index" do
            my_question = FactoryGirl.create(:question, :user_id => @test_user.id)
            post :destroy, {:id => my_question.id}
            expect(response).to redirect_to(questions_path)
          end

          it "destroys a record" do
            my_question = FactoryGirl.create(:question, :user_id => @test_user.id)
            post :destroy, {:id => my_question.id}
            expect(Question.all).not_to include my_question
          end

          it "displays a flash message" do
            my_question = FactoryGirl.create(:question, :user_id => @test_user.id)
            post :destroy, {:id => my_question.id}
            expect(flash[:success]).to eq("You have successfully deleted the question")
          end
        end
      end
    end

This spec runs three separate tests for Questions#destroy, but its not very DRY. For each test (called examples in Rspec parlance) we need to generate a question belonging to the user and then make a post request to destroy it. I literally copied and pasted the first two lines of each example. Fortunately there are two methods to help us avoid this repetition.

##let

`let` is a way to lazily define a variable that we can call as many times as we like. So instead of having to make a `FactoryGirl` call for each example, we can just define it once. The variable is loaded into memory when we actually call it, not when its defined. That changes our code to:

    describe "POST destroy success" do
      #should be able to destroy when the post belongs to you
      #or if you are an admin
      ControllerMacros::ALL_ROLES.each do |r|
        context "as #{r}" do
          login_as r
          let (:my_question) { FactoryGirl.create(:question, :user_id => @test_user.id) }

          it "redirects to index" do

            post :destroy, {:id => my_question.id}
            expect(response).to redirect_to(questions_path)
          end

          it "destroys a record" do
            post :destroy, {:id => my_question.id}
            expect(Question.all).not_to include my_question
          end

          it "displays a flash message" do
            post :destroy, {:id => my_question.id}
            expect(flash[:success]).to eq("You have successfully deleted the question")
          end
        end
      end
    end

##subject
And it gets even better. `subject` gives us a block of code that we can define once and then call when ever we like. The line `post :destroy, {:id => my_question.id}` is a perfect candidate for that. Let's alter our code again like so
 
    describe "POST destroy success" do
      #should be able to destroy when the post belongs to you
      #or if you are an admin
      ControllerMacros::ALL_ROLES.each do |r|
        context "as #{r}" do
          login_as r
          let (:my_question) { FactoryGirl.create(:question, :user_id => @test_user.id) }
          subject(:post_destroy) { post :destroy, {:id => my_question.id} }

          it "redirects to index" do
            post_destroy
            expect(response).to redirect_to(questions_path)
          end

          it "destroys a record" do
            post_destroy
            expect(Question.all).not_to include my_question
          end

          it "displays a flash message" do
            post_destroy
            expect(flash[:success]).to eq("You have successfully deleted the question")
          end
        end
      end
    end

Much more DRY! There's less typing, and as an added benefit the implementation details are a bit more abstracted. If someone wants to read my tests they can tell with a little less effort what I am up to. `post_destroy` is hopefully pretty clear about what is going on. If someone wants to know specifically how I did it they can glance up to where I defined `subject`, but otherwise they can just assume I did it the standard way and move on.
