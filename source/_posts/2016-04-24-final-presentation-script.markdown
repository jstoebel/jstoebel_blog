---
layout: post
title: 'Final Presentation: script'
date: '2016-04-24T17:20:11+00:00'
permalink: final-presentation-script
---
Here's my script for my final presentation.

##Slide 1: CodeWorkout homepage

 - CodeWorkout is an open source Rails project originating out of Virginia Tech. Its purpose three fold:
 - to help new programmers improve their skills with short programing workouts
 - to provide a learning platform for professors to host in class assignments and be able to easily assess their student's progress
 - data collection on the behavior of new programmers and how they approach challenging problems.

##Slide 2: Google Slide: Our team's project

 - I was a member of a six person team in my class: Software Engineering at Berea College. We chose CodeWorkout as our project for the semester with the hope to make a meaningful contribution to the project.
 - The project lead, Steve Edwards charged us to created a Q&A forum for the site. They wanted something similar to Stackoverflow where students could ask questions when they got stuck and others could post possible solutions.

## Slide 3: Google Slide: My contribution: unit tests
We needed:

 - tools and workflow to stay in sync with each other. 
 - to make sure that the expectations of our project weren't broken after merging in new code.
 - demonstrate to the team lead that we had thought through our design thoroughly.
 - have a relatively smooth hand off of our work to the team lead.

Fortunately a strong set of unit tests for our work were able to help with all of these needs.

## Slide 4: Example of Rspec test.

 - Rspec is a comprehensive Ruby testing framework with excellent integration for Rails.
 - It is easy to read. As you can see here the expectations for the test shown here are clearly spelled out. If you want to dig into the implementation of each test, that is easy enough to read as well. They are more than just tests. If you want to know about the expectations of a piece of software just read its specs!
 - Rspec is extremely DRY (don't repeat yourself). If you have multiple tests on the same block of code, Rspec provides some great tools for defining the commonalities across multiple tests that you can then reference when ever you like.

##Slide 5: Sublime

The following tabs are open

 - questions\_controller.rb#update
 - questions\_controller\_spec.rb -> POST update success
 - controller\_macros

 - As an example, let's look at a single user story: 

        Users should be able to update a question they have posted.

(OPEN QUESTIONS CONTROLLER, UPDATE ACTION)

 - When a request comes in to this controller action: 
     - the appropriate question is pulled from the database
     - we authorize the user's permission to update the question.
     - the posted parameters are assigned to the Active Record object.
     - Assuming the save is successful, the user is redirected and a confirmation message is displayed.
     - If the save was not successful due to bad params, the form to edit the question is rendered and an error message is displayed.

(OPEN TEST)

 - Here is the set of unit tests (or specs in Rspec parlance) that I've defined for the questions controller.
 - The first thing to note here is this before block at the top. This allows me to define any code that should be run before any individual test.
 - In this set of tests, we are using the FactoryGirl gem to populate the necessary database tables. Once a test is done, the database is rolled back and we repeat the process for the next test. This is ensure that when one test alters the state of the database in one test it will not have an unintended consequences on the following tests.
 - Scrolling down (line 292) here is the Spec in question.
 - This update action requires a user be signed in, which means that we need a way to simulate the logging in of an appropriate user. To make things more complicated, CodeWorkout has three types of users, admins, instructors and regular_users (students), each with differently defined abilities. In order to keep things, DRY we should ideally be able to iterate over each user role, identify a user with that role, and then run tests as that user.
 - CodeWorkout uses the Devise gem, a popular framework for authentication. Devise comes with some helper methods for testing logging in a user.

(OPEN CONTROLLER MACROS)

 - I've defined a method `login_as` that accepts a role name as a string. When the method is called a user is found in the database, matching that role and devise logs them in. We save that user record as an instance variable so we have access to it later.
 - Once that user is logged in, we are going to define some parameters. The method `let` allows us to define variables that are lazy loaded, meaning that Ruby won't actually execute them until they are needed. 
 - Here I create a new question using Factory Girl that belongs to the user I just logged in.
 - Then I define a new hash to represent a parameter on the question I want to update. Specifically, I am changing the question's title to the string `"new title"`. I then merge that change into a full set of parameters to be posted to the update action.
 - Finally I initiate the HTTP post request. Just like `let` lets us lazyily define a variable, `subject` lets us lazily define a block of code that is run when ever we call it.
 - The whole point of this is that I've done all of my set up once. For each test inside this spec I simply call `post_update` and the http request is made.
 - Now I simply make assertions to test each expectation of the controller action. I expect:
    - that the user will be redirected to the show page of the related question.
    - that all of the parameters I expected to change will match the parameters of the resulting question that was pulled and updated. I do this by iterating over the hash `change` and ensuring that each of its values match the coorsponding values in the ActiveRecord object `question` that was created in the action. 
    - and finally, I want to test that the correct  confirmation message is displayed.

 - All of this should constitute a full set of expectations for how the "happy path" of this controller action should run. But what about the "sad path", or the code that runs when a user, for example does not have permission to update a question? I've created a separate spec for this.
 - The exciting thing here is the set up is almost exactly the same. The only difference is that I define a new user (who will for sure not be the one logged in) and then create a question for them. Then I attempt to change the record in the exact same way.
 - This time I expect for the request to cause an raised exception at the authorization layer due to the question not belonging to the user. Specifically, the user should be redirected to the home page and an error message should be displayed.

(SWITCH TO COMMAND LINE)

 - When I'm ready, I can run my tests from the command line. Each of those green dots represents a successful test run. 
 - If tests fail, or cause an error, I am given a report with a traceback making it relatively painless to debug.
 - And that's it! So in conclusion, a test suite gives us the ability to define every expectation we have on a program. When the program changes, we use the test suite to assure that no expectations are broken before we check in our changes into source control.
