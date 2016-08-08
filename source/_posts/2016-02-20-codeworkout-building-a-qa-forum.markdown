---
layout: post
title: 'CodeWorkout: building a Q&A forum'
date: '2016-02-20T22:17:27+00:00'
permalink: codeworkout-building-a-qa-forum
---
[CodeWorkout's][1] project lead, Steve Edwards has given us all a good direction to contribute towards the project: a Q&A forum where students can seek help from their peers similar to Stackoverflow.

We've been advised to rapidly develop a version 0, which includes the most basic version of a Q&A service and then add on to it later. I imagine this as a very basic [CRUD][2] which is as follows

 - User's may post new questions to the forum and edit their own questions 
 - User's may post responses to questions.
 - User's may view questions posted by anyone.
 - Admin users may remove questions or responses.


This feature will be completely new to the code base meaning that we will need to add the following:

 - A migration file to add two new tables to the database:  `questions` and `responses`
 - a controller for `questions` and `responses` with actions for `index`, `show`, `new`, `create`, `edit`, `update`, `delete` and `destroy`.
 - Models for `questions` and `responses`
 - Views for all of the above actions.


Additionally we will need to edit `app/models/ability.rb` to specify that student's may edit or destroy only their own `questions` and `responses` and admins may edit and destroy any post.



  [1]: https://github.com/web-cat/code-workout
  [2]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
