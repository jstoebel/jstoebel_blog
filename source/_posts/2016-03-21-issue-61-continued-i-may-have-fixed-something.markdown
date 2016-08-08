---
layout: post
title: 'Issue 61 continued: I may have fixed something!'
date: '2016-03-21T01:19:27+00:00'
permalink: issue-61-continued-i-may-have-fixed-something
---
After several more hours looking into issue 61 ([see previous blog post][1]) I may have found a solution for the problem! It was a multi-step process:

## WorkoutOffering and WorkoutPolicy

The sample data (as provided by `rake db:populate`) was providing a `Workout`, but for some reason it was not showing up when I clicked on the courses page. Here's why: A Workout is a collection of exercises. Workouts can belong to the gym (anyone can do them at any time) and they can belong to any number of courses. The record that links a `Workout` to a specific course is called a `WorkoutOffering`. If we look at the view courses#show we find the following:

      - @course_offerings.each do |offering|
        - if @is_student
          - offering.workout_offerings.visible_to_students.in_groups_of(2, false) do |row|
            = render row, locals: { uid: current_user.id }
            .clearfix
        - else
          - offering.workout_offerings.in_groups_of(2, false) do |row|
            = render row, locals: { uid: current_user.id }
            .clearfix 

The method `visible_to_students` is filtering a course's workout_offerings based on if they should be visible to students. This is defined as:

WorkoutOffering needs to have 

 - `published = true`
 - `opening_date < DateTime.now` 

Also, each `WorkoutOffering` follows a `WorkoutPolicy`. That policy must have `invisible_before_review = false`


Currently `rake db:populate` isn't creating any `WorkoutOffering` or `WorkoutPolicy` objects. Let's create them now using factory girl.

    FactoryGirl.define do
      factory :workout_offering do
        course_offering_id 1
        workout_id 1
        published true
        opening_date DateTime.now
        soft_deadline 1.year.from_now
        hard_deadline 1.years.from_now
      end
    end

and...

    FactoryGirl.define do
      factory :workout_policy do
        name "my policy"
        invisible_before_review false

        after(:create) do |wpolicy|
            create(:workout_offering, workout_policy: wpolicy)
        end
      end
    end

the method `after` is callback, meaning that once we make a `WorkoutPolicy` record we will run the `workout_offering` factory and associate that newly created policy with it.

##Display feedback if closed

On the view `exercise#practice` where exercises are displayed, we will display the exercise feedback rather than the exercise itself if the workout is closed. So how do we know if it is closed? This is a somewhat complex question since a WorkoutOffering has both a soft and hard deadline _and_ individual students may request extensions for an offering. Ultimately, I decided that the phrasing _completely closed_ should mean the offering is closed for everyone in the class. Fortunately there is an instance method to help with this. Calling the method `ultimate_deadline` returns the date by which the workout is due for every student in the class. So if we switch over to the exercise#practice view we see:

    - @exercise_version.prompts.each do |question_prompt|
        - prompt = question_prompt.specific
        #display each prompt in the exercise here...

Its simply a matter of adding a conditional statement to check if the prompt should be shown, or feedback should be displayed.

    - @exercise_version.prompts.each do |question_prompt|

      / If the workout deadline hasn't passed:
      - if @workout_offering.ultimate_deadline >= DateTime.now
          #render the prompt
      - else
          #render feedback

As of this writing, I still want to clean up my solution, add comments and make the HTML look decent, but the jist of the solution is here!


  [1]: http://www.jstoebel.com/blog/codeworkout-ticket-issue-61/
