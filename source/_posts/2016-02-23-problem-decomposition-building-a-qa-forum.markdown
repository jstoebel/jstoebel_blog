---
layout: post
title: 'Problem Decomposition and Components: building a Q&A forum'
date: '2016-02-23T01:06:06+00:00'
permalink: problem-decomposition-building-a-qa-forum
---
In my [previous post][1] I talked about how I see us approaching our version 0 of a Q&A forum. In this post I will go into more detail on how I see it implemented:

##Question Model
A Question object will need the following attributes

- id (auto incrementing)
- text (string) the text of the question being asked.
- user_id: the foreign key to the user table
- created_at: the date the question was created.
- updated_at: the date the question was last updated

##Response model

A response object will need the following attributes

- id (auto incrementing)
- text (string) the text of the response.
- user_id: the foreign key to the user table
- question_id: the foreign key to the question table
- created_at: the date the question was created.
- updated_at: the date the question was last updated

## QuestionController

The Question controller will need the following actions

- index: pull all records from the database
- show: pull one record from the database for display
- new: instantiate a new record
- create: accept post params for creation of new object. Save to database if valid. Re render new if not valid
- delete: pull record to be deleted (we could also do this as a JS popover)
- destroy: destory object

##ResponseController

We might not need a response controller at all. All requests against the Response resource might be made within the context of viewing them within their respective question.

## Views

We will need the following views

- Question#index
- Question#show
- Question#new
- Question#delete (or maybe not if we use a popover)

##Routes
We will need to edit the routes file to allow for requests to reach each of our actions. The best way to do this is probobly using [RESTful routes][2].


  [1]: http://CodeWorkout:%20building%20a%20Q&A%20forum
  [2]: http://guides.rubyonrails.org/routing.html
