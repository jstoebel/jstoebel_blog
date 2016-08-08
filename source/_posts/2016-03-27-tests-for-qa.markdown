---
layout: post
title: Tests for Q&A
date: '2016-03-27T14:11:58+00:00'
permalink: tests-for-qa
---
For the next phase of the Q&A forum, I will be writing the tests for everything in Rspec. Here is what I believe, according to the design document and a few (safe?) assumptions. 

##Controllers

###Questions

####index
 - return 200
 - pull all questions in database.
 - pull all of question's responses
 - render the index view

####show
 - return 200
 - pull expected question
 - render show view
 - instantiate new response record

####new
 - return 200
 - instantiate a new Question.
 - render the new view

####create (good params)
 - created new question per given params
 - display flash message
 - redirect to index

#### create (bad params)
 - render new form
 - display flash message
 - don't save record
 - return 200

####edit

 - return 200
 - pull expected record
 - render edit view

#### update (good params)
 - updates record as expected
 - display flash message
 - redirect to index

####update (bad params)
 - render edit view
 - display flash message
 - don't save record
 - return 200

####delete
 We don't have a delete controller. Skip!

####destroy
 - destroy right record
 - redirects to index
 - displays flash message

###Responses


####index
don't have one. Skip!

####show
 - pull the right record
 - render show view
 - return 200

####new
don't have one!

####create (good params)
 - create response according to params
 - display flash message
 - redirect to question#show

####create (bad params)
 - display message
 - don't save record
 - render question#show

####edit
 - pull expected record
 - render edit view

####update (good params)
 - update response according to params
 - display flash message
 - redirect to question#show

####update (bad params)
 - display message
 - don't save record
 - render question#show

**TODO**: once we write cancan abilities, we need to test all possible outcomes for all roles.  


##Models

###Questions
 - test validations for `:title` and `:body`

###Responses

 - test validation for `:text`

##Routes
 Need to test all of the routes defined in both questions and responses

##Views
####Questions#index
 - questions displayed
 - new link
 - edit 

###Questions#new
 - submit button

###Questions#edit
 - submit button

###Questions#_field
 - title
 - body
 - tags
 - exercise_id

###Questions#show
 - delete button
 - question displayed
 - form for new response

###Responses#edit
 - form to edit

###Responses#show
 - attributes for response shown
 - delete button
 - edit link
##quet
