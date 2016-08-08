---
layout: post
title: 'Software Engineering Portfolio: an outline'
date: '2016-04-20T23:28:32+00:00'
permalink: software-engineering-portfolio-an-outline
---
For my portfolio and final presentation in Software Engineering I will be show casing my work with Rspec, the testing framework for Rails. My general idea is to create a screen cast that walks my audience through the tests I created (Rspec) as well as how I interacted with other parts of this Rails project like Devise (for authentication), Cancancan (for authorization) and FactoryGirl (for creating test data). I'll be showcasing my work as

 1. a portfolio (really a series of web pages), and,
 2. as a screen cast which I will record for my portfolio and also present live during our final.

Here is my rough draft outline of the project.

##The portfolio

 1. An introduction to CodeWorkout and our teams charge to develop a Q&A forum.
 2. The problem, specifically:
        - working with in a team of 4 and later 6. Need tools and workflow to stay on sync with each other.
        - need to work with the intention of an easy enough hand-off of our contribution into the project.
        - Need to demonstrate to the team lead that we have thought through our design thoroughly.
        - All of this lead me to see the necessity for contributing unit tests for our Q&A forum. If we have tests we can check them prior to each commit. If the tests pass, we know that our changes haven't broken any expectations we have for our code base.
 3. The design of my unit tests: What assertions I made for each Rspec test. This part will actually be easy since each test is named after the basic assumption it is testing.
 4. The implementation: Obviously I can direct readers to my Github page. Here would also be a great time to talk about some of the neat tricks I learned like
       - How to interact with Devise
       - How Rspec + FactoryGirl == <3
       - How authorization fits into this.
       - Other neat tricks I learned about Rspec.
 5. Where I see this project going in the near future.


## The presentation
My presentation will be a walk trough of slides and a walk through of the code I wrote. The basic structure will be an abridged version of the above.
