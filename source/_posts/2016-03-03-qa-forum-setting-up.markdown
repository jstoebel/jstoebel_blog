---
layout: post
title: 'Q&A forum: setting up'
date: '2016-03-03T22:36:07+00:00'
permalink: qa-forum-setting-up
---
Following up from my last post, here is a walk through on generating all of the required components for our Q&A forum.

##Questions

First, let's generate all of the stuff we need related to Q&A questions. This will include the controller, model, migration, views and modification of the routes file.

###QuestionsController

The controller handles all of the incoming requests for the resource and renders the appropriate response. Just to get things started let's make our controller able to handle all actions in CRUD.

    $ rails generate controller questions index show new create edit update delete destroy
      create  app/controllers/questions_controller.rb
       route  get 'questions/destroy'
       route  get 'questions/delete'
       route  get 'questions/update'
       route  get 'questions/edit'
       route  get 'questions/create'
       route  get 'questions/new'
       route  get 'questions/show'
       route  get 'questions/index'
      invoke  haml
      create    app/views/questions
      create    app/views/questions/index.html.haml
      create    app/views/questions/show.html.haml
      create    app/views/questions/new.html.haml
      create    app/views/questions/create.html.haml
      create    app/views/questions/edit.html.haml
      create    app/views/questions/update.html.haml
      create    app/views/questions/delete.html.haml
      create    app/views/questions/destroy.html.haml
      invoke  rspec
      create    spec/controllers/questions_controller_spec.rb
      create    spec/views/questions
      create    spec/views/questions/index.html.haml_spec.rb
      create    spec/views/questions/show.html.haml_spec.rb
      create    spec/views/questions/new.html.haml_spec.rb
      create    spec/views/questions/create.html.haml_spec.rb
      create    spec/views/questions/edit.html.haml_spec.rb
      create    spec/views/questions/update.html.haml_spec.rb
      create    spec/views/questions/delete.html.haml_spec.rb
      create    spec/views/questions/destroy.html.haml_spec.rb
      invoke  helper
      create    app/helpers/questions_helper.rb
      invoke    rspec
      create      spec/helpers/questions_helper_spec.rb
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/questions.js.coffee
      invoke    scss
      create      app/assets/stylesheets/questions.css.scss


This action creates several new files for us:

* The controller itself
* routes for each action we requested 
* a new directory of views for the questions resource
* test files for the questions resource
* static assets to be used inside the resource (Coffescript and SCSS)

The file of most immediate concern is the controller. Taking it a look, it contains all of the actions we specified.

     class QuestionsController < ApplicationController
      def index
      end

      def show
      end

      def new
      end

      def create
      end

      def edit
      end

      def update
      end

      def delete
      end

      def destroy
      end
    end

These actions won't do anything other than render their appropriate view, but its a good start. Next the routes file:

##Routes

Looking in `config/routes.rb` we see that new routes were created for us to reach each of these controller actions. 

    get 'questions/index'

    get 'questions/show'

    get 'questions/new'

    get 'questions/create'

    get 'questions/edit'

    get 'questions/update'

    get 'questions/delete'

    get 'questions/destroy'


These work ok, but a much better way is to implement [RESTful routes][1].  This is preferable because they are 1) easier to organize 2) more concise and 3) provide common sense HTTP verbs for each CRUD action. To implement this, let's delete all of the above and replace it with simply `resources :questions`. This simple method call creates all of the routes we will likely need to service the questions resource. We can take a look at the actual paths that were added by using `rake routes`

    $ rake routes | grep questions
       questions_index GET      /questions/index(.:format)                                                                              questions#index
                                 questions_show GET      /questions/show(.:format)                                                                               questions#show
                                  questions_new GET      /questions/new(.:format)                                                                                questions#new
                               questions_create GET      /questions/create(.:format)                                                                             questions#create
                                 questions_edit GET      /questions/edit(.:format)                                                                               questions#edit
                               questions_update GET      /questions/update(.:format)                                                                             questions#update
                               questions_delete GET      /questions/delete(.:format)                                                                             questions#delete
                              questions_destroy GET      /questions/destroy(.:format)                                                                            questions#destroy


Sweet! Now our app has URLs that will route to the related action in the questions controller.

## Model

Now we need to create the model which is an API for the corresponding database table.

    $ rails generate model Question      invoke  active_record
      create    db/migrate/20160303224849_create_questions.rb
      create    app/models/question.rb
      invoke    rspec
      create      spec/models/question_spec.rb
      invoke      factory_girl
      create        spec/factories/questions.rb

Our model is created. There is a lot we can put in there, but the first thing should be to tell the model how it associates with other models. 

    class Question < ActiveRecord::Base
      has_many :responses
      belongs_to :user
      belongs_to :exercise
    end

The Question class now has three additional methods: `responses`, `user` and `exercise` which all grab the records you would expect. With all of this stuff using ActiveRecord we might not need to type SQL at all!

#Migration 
This provides a file we can add ruby code to which will add the actual database table. Let's do that now:

    class CreateQuestions < ActiveRecord::Migration
      def up
        create_table :questions do |t|
            t.string :title
            t.string :body
            t.string :tags
            t.integer :user_id
            t.integer :exercise_id
            t.timestamps
        end
        add_foreign_key :questions, :users
        add_foreign_key :questions, :exercises

       end

      def down
        drop_table :questions
      end
    end

This migration file has two methods, `up` and `down`. When we are running `rake db:migrate` the `up` method will run. When we run `rake db:rollback` the `down` method responds. `down` should always reverse what `up` does so programmers can move backwards and forwards in the code migrations. With the migration written we can run it:

    $ rake db:migrate
    == 20160303224849 CreateQuestions: migrating ==================================
    -- create_table(:questions)
       -> 0.0016s
    -- add_foreign_key(:questions, :users)
       -> 0.0000s
    -- add_foreign_key(:questions, :exercises)
       -> 0.0000s
    == 20160303224849 CreateQuestions: migrated (0.0018s) =========================

It works! It's not a bad idea to make sure the down method works to.

    $ rake db:rollback
    == 20160303224849 CreateQuestions: reverting ==================================
    -- drop_table(:questions)
       -> 0.0007s
    == 20160303224849 CreateQuestions: reverted (0.0008s) =========================

Great! Don't forget to migrate back up once you've done this. An important note here is that an actual database table is being dropped meaning that actual data can be lost. No confirmation at all! Fortunately we are not in charge of production data so there isn't a danger for us to worry about.

##Repeat for responses

Repeat all of the above for the responses resource and we have set up our infrastructure.

##Conclusion
It takes a bit of work to get a new part of an app set up, but once we've done so, we are ready to start breaking up the work. You can see the work I've done [here][2]


  [1]: http://guides.rubyonrails.org/routing.html
  [2]: https://github.com/jstoebel/code-workout/tree/qa_setup_jacob
