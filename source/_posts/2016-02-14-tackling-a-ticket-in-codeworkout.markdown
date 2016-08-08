---
layout: post
title: Tackling a ticket in Codeworkout
date: '2016-02-14T20:50:50+00:00'
permalink: tackling-a-ticket-in-codeworkout
---
Today I am looking at a ticket in the CodeWorkout issue tracker. The issue is already being worked on by someone else so I won’t implement it but it seemed like a fun way to dive into the code base. The ticket is:

> #### <span class="js-issue-title">Randomize the workout exercises</span> <span class="gh-header-number">#75</span>
> 
> Implement the randomization of the order of the exercises in a workout. Might need to do assign a random order of questions to each user.

In addition to being a tool for teaching people computer programming, CodeWorkout is also a platform for collecting data on new programmers and how they problem solve. Any valid experiment needs the ability to randomize the order to conditions, thus the ticket. Let’s give it a shot.

Firing up the rails console, we see that there is one Workout included in the sample data. If we go into the course CS 1114: Introduction to Software Design we don’t see any workouts. What gives? Studying the ERD, we’re able to trace a workout to a course as follows:

*   A workout has many workout offerings. A WorkoutOffering represents how a workout is presented to a unique course offering.
*   A CourseOffering has numerous WorkoutOfferings (aka homework assignments or problem sets).
*   A Course has many CourseOfferings (aka numerous sections of a single course).

All of this is a bit complicated, but it gives instructors the ability to assign a workout to their students with deadlines independent of other sections. We need to complete a new WorkoutOffering object in order to hook up our Workout with the CourseOffering:

     
    co = CourseOffering.first  
    => #<CourseOffering id: 1, course_id: 1, term_id: 1, label: "MWF 10:00am", url: "http://courses.cs.vt.edu/~cs2114/Fall2013", self_enrollment_allowed: false, created_at: "2016-02-14 18:27:46", updated_at: "2016-02-14 18:27:46", cutoff_date: nil>`
    
    wkout = Workout.first  
    => #<CourseOffering id: 1, course_id: 1, term_id: 1, label: “MWF 10:00am”, url: “[http://courses.cs.vt.edu/~cs2114/Fall2013&#8221](http://courses.cs.vt.edu/~cs2114/Fall2013&#8221);, self_enrollment_allowed: false, created_at: “2016-02-14 18:27:46”, updated_at: “2016-02-14 18:27:46”, cutoff_date: nil>
    
    #lets make a new workout offering to connect the workout to the course offering  
    wo = WorkoutOffering.new  
    => #<CourseOffering id: 1, course_id: 1, term_id: 1, label: “MWF 10:00am”, url: “[http://courses.cs.vt.edu/~cs2114/Fall2013&#8221](http://courses.cs.vt.edu/~cs2114/Fall2013&#8221);, self_enrollment_allowed: false, created_at: “2016-02-14 18:27:46”, updated_at: “2016-02-14 18:27:46”, cutoff_date: nil>
    
    #opened yesterday and closing 100 days from now  
    wo.assign_attributes({course_offring_id: co.id, workout_id: wkout.id, opening_date: Date.today-1, hard_deadline: Date.today + 100, published: true })  
    => #<CourseOffering id: 1, course_id: 1, term_id: 1, label: “MWF 10:00am”, url: “[http://courses.cs.vt.edu/~cs2114/Fall2013&#8221](http://courses.cs.vt.edu/~cs2114/Fall2013&#8221);, self_enrollment_allowed: false, created_at: “2016-02-14 18:27:46”, updated_at: “2016-02-14 18:27:46”, cutoff_date: nil>
    
    wo.save

Now we go back to the course and we see the workout appears in the course! Now that we understand how a workout is hooked into a course we need to figure out how to randomize. My guess is that it isn’t enough to simply randomize the ordering of exercises each time we display them in the workout. That would result in a user seeing a different ordering each time they index the workout. Most likely, we want user to be given the exercises in a random order and that order to persist each time they pull up the workout. This could be achieved in a number of ways, but one potential way of accomplishing this would be to create a new table that stores this order. The table workout_scores, holds an instance of a single progress on a workout. We could potentially associate with this table a new table called workout_ordering that maps each exercise in the workout to an auto incrementing number. The first time a user attempts a workout, this ordering is created, but when the user returns to the workout later, the original ordering is referenced to determine the ordering the exercises are displayed.

Since this ticket is claimed by someone else, I will not work on implementing it so as not step on toes. Regardless, this was a useful exercise in understanding the codebase a little more.
