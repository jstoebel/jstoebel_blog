---
layout: post
title: 'CodeWorkout ticket: issue 61'
date: '2016-03-17T00:06:13+00:00'
permalink: codeworkout-ticket-issue-61
---
Today I am taking a look at an [open ticket][1] for CodeWorkout. It is as follows:

> Once a workout is completely closed, make available the optimal answer code to the exercises written by the instructors available for students.

This seems like something I could handle with a bit of work. Exploring the code base's models in depth, we see that each exercise is associated with an answer as follows:

 - an `Exercise` has many `ExerciseVersion`s The attribute `currrent_version` gets you the most recent one.
 - a version has one or many `prompts`
 - Each `Prompt` has an attribute `feedback` which contains the most optimal answer. 

The basic idea, in my mind, would include the following changes:

 - Add an attribute to `Workout` called `open`.
 - When showing a workout (/gym/workouts/1 )if a workout is closed, label it as such.
 - When users view an exercise in a workout that is closed, rather than accepting an attempt, instead display the feedback.

##Edit


While I was working on this ticket, I found that the test Workout has closed! I'm not exactly sure why, but earlier I found that that Workout reported "a few minutes left" to complete as a deadline. After more looking around I found the model `WorkoutOffering` with attributes `opening_date` `soft_deadline` and `hard_deadline`. This would seem to be related to the problem (hard_deadline is being set to close to creation date)  but it turns out there are no WorkoutOfferings in the database to begin with. `WorkoutOffering`s are used when a `Workout` is tied to a course, and this `Workout` was in the gym. 
There might be a bug going on here but this further exploration has revealed to me that my original approach for the issue is off. It turns out that a WorkoutOffering is something that can be closed, not a Workout itself since Workouts can be used in multiple courses

#### For further exploration

 - Create a `WorkoutOffering` using FactoryGirl.
 - Set a due date,  see what happens once it passes.
 - Define what exactly is meant by "completely closed" (see my question on this ticket)
 - Revise the exericse#show view to display the feedback when the offering is closed.

Also: I seem to have stumbled upon two possible bugs. 

#### Gym workout as admin
First  when trying to view a Workout in the gym as admin, the Workout becomes due after a few minutes. Once the Workout is due, I get an error when trying to view it


      undefined method `time_limit_for' for nil:NilClass
     app/models/workout_score.rb:91:in `closed?'
    app/models/workout_score.rb:123:in `show_feedback?'
    app/views/workouts/_workout.html.haml:58:in `_app_views_workouts__workout_html_haml__2106449378163807726_70138070525280'
    app/views/workouts/index.html.haml:11:in `block in _app_views_workouts_index_html_haml___577289284953895515_70138077612680'
    app/views/workouts/index.html.haml:10:in `_app_views_workouts_index_html_haml___577289284953895515_70138077612680'
  [1]: https://github.com/web-cat/code-workout/issues/61

##Gym workout as student

Second, when trying to start a workout in the gym as a student I am redirected to the home page with an error message "You are not authorized to access that page." I might want to check the Cancan abilities for this one.
