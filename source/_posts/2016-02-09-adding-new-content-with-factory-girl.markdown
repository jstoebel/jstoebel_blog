---
layout: post
title: Adding new Content With Factory Girl
date: '2016-02-09T01:52:40+00:00'
permalink: adding-new-content-with-factory-girl
---
CodeWorkout uses the gem `factory_girl` to seed the database with test data and then provides a rake task (`rake db:populate)` to allow users to reset the database to this beginning state.

Factory Girl appears somewhat complex at first glance, but its worth the effort to try and understand. In Factory Girl, sets of data are defined inside factories, and then called as many times as we wish. For example, take a look at this simple example from the creator’s Github page  

    FactoryGirl.define do  
      factory :user do  
      first_name "John"  
      last_name "Doe"  
      admin false  
    end

Here, we are creating one factory called :user. Whenever we fire up :user, by calling for example FactoryGirl.create(:user) a :user object is written to the database. We can also pass in our own attributes as well to create some variety.

Ok, that was a simple example. CodeWorkout is much more complex. Here is part of the file spec/factories/exercises.rb.

    FactoryGirl.define do
          sequence :exercise_no, 1
          factory :exercise do
            ignore do
              num { generate :exercise_no }
            end
            external_id { 'E' + num.to_s }
            name { 'Factorial ' + num.to_s }
            question_type { Exercise::Q_CODING }
            is_public true
            experience 50

        # tags: Java, factorial, multiplication, function

        factory :coding_exercise do
          ignore do
            creator_id 1
            question "Write a function in Java called `factorial()` that will "\
              "take a\npositive integer as input and returns its factorial as "\
              "output.\n"
            feedback "Explanation for the correct answer goes here.  This is "\
              "just\nan example.\n"
            class_name 'Factorial'
            method_name 'factorial'
            starter_code "public int factorial(int n)\n"\
              "{\n"\
              "    ___\n"\
              "}\n"
            wrapper_code "public class Factorial\n"\
              "{\n"\
              "    ___\n"\
              "}\n"
            test_script ""\
              "0,1\n"\
              "1,1\n"\
              "2,2\n"\
              "3,6\n"\
              "4,24,hidden\n"\
              "5,120,hidden\n"
          end

          question_type { Exercise::Q_CODING }
          language_list 'Java'
          tag_list 'factorial, function, multiplication'
          style_list 'code writing'

          after :create do |e, v|
            e.current_version = FactoryGirl.create :exercise_version,
              exercise: e,
              creator_id: v.creator_id
            e.exercise_versions << e.current_version
            FactoryGirl.create :coding_prompt,
              exercise_version: e.current_version,
              question: v.question,
              feedback: v.feedback,
              class_name: v.class_name,
              method_name: v.method_name,
              starter_code: v.starter_code,
              wrapper_code: v.wrapper_code,
              test_script: v.test_script
            e.save!
          end
        end


Ok there’s a LOT to take in here. But here’s the gist of what’s going on.

1.  We create a sequence called :exercise_no which is just a generator. Each time we call it we get the number one more than the last. This lets us number our exercises form 1 to however high we want.
2.  We defined a number of attributes for an exercise.
3.  When we get to `factory :coding_exercise` we begin defining attributes for related entities. Attributes like question and feedback belong to the CodingPrompt class
4.  The `after :create do |e, v|` is a call back, or function that calls automatically once an object is created. Basically, once we create an Exercise, we then create the corresponding entities ExerciseVersion and CodingPrompt

The thing that brings this all together is the rake task db:populate (which we used to set up the database.) We can find it in `lib/tasks/sample_data.rake`. Hopefully there is some intuition of what is going on here:


    namespace :db do
      desc "Reset database and then fill it with sample data"
      task populate: [:environment, :reset] do
        FactoryGirl.create(:organization)
        FactoryGirl.create(:term)
        FactoryGirl.create(:term2)
        FactoryGirl.create(:course)
        c = FactoryGirl.create(:course_offering)
        FactoryGirl.create(:course_offering2)

    FactoryGirl.create(:course_enrollment,
      user: FactoryGirl.create(:admin),
      course_offering: c,
      course_role: CourseRole.instructor)
    FactoryGirl.create(:course_enrollment,
      user: FactoryGirl.create(:instructor_user,
        first_name: 'Ima',
        last_name:  'Teacher',
        email:      "example-1@railstutorial.org"),
      course_offering: c,
      course_role: CourseRole.instructor)
    50.times do |n|
      FactoryGirl.create(:course_enrollment,
        user: FactoryGirl.create(:confirmed_user,
          first_name: Faker::Name.first_name,
          last_name:  Faker::Name.last_name,
          email:      "example-#{n+2}@railstutorial.org"),
        course_offering: c)
    end

    # Create a workout with one exercise, and a second exercise
    FactoryGirl.create :workout_with_exercises
    FactoryGirl.create :coding_exercise, name: 'Factorial 3'
    end


We march down the script, creating entities as we go. We see exercises created on the last two lines. `FactoryGirl.create :workout_with_exercises` creates a Workout object with some related exercises (find that in `spec/factories/workouts.rb`) and then our own factory, :coding_exercise, which creates the content we just went over.

Sweet! Next step, lets drop our own content into a new factory and add it to the rake task to let user’s generate a brand new shiny Python Workout!
