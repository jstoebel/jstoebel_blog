---
layout: post
title: CodeWorkout Q&A forum, creating and editing a question
date: '2016-03-12T19:57:11+00:00'
permalink: codeworkout-qa-forum-creating-and-editing-a-question
---
Over spring break, I added the features to create and edit a question in our new CodeWorkout Q&A forum. Here's a summary of that work:

##Controller Actions

Here's the work I added to the relevant controller actions:

    def new
      #pre:
        #exercise_id (optional): the exercise this question should be associated
        #with
      #post: a new question object is instantiated but not saved
        #question#new is rendered

      @question = Question.new({
        :exercise_id => params[:exercise_id]
        })

    end

    def create
      #pre:
        #params: the parameters to be used to create this question
      #post: 
        #a new question object is saved -> redirect to index
        #OR new question is not saved -> render new
      @question = Question.new(safe_assign)
      @question.user_id = current_user.id

      if @question.save
        flash[:success] = "Question saved!"
        redirect_to questions_path
      else
        flash[:error] = "Error creating your question."
        render 'new'
      end

    end

    def edit
      #pre:
        #id: the id of the question to edit
      #post:
        #edit view is rendered
      @question = Question.find(params[:id])
    end

    def update
      #pre:
        #params: question attributes to be assigned
      #post
        #the question object is saved -> redirect to index
        #OR new question is not saved -> render edit
      @question = Question.find(params[:id])
      @question.assign_attributes(safe_assign)

      if @question.save
        flash[:success] = "Question saved!"
        redirect_to questions_path
      else
        flash[:error] = "Error updating question."
        render 'edit'      
      end
    end

    private
    def safe_assign
      params.require(:question).permit(:title, :body, :tags, :exercise_id)
    end


The flow of control works like this: when a user makes a request for `/questions/new` they hit the action `new`. In that action we make a new `Question` object and (implicitly) render `new.html.haml`. Also, if an exercise_id was passed into params we add that to the question (more on this later).

###The View

Here is `new.html.haml` 

    =link_to questions_path do
        .btn.btn-default
            %span.glyphicon.glyphicon-arrow-left
                Back

    %h1 Create a new Question

    =render(:partial => 'application/record_error_messages', :locals => {:record => @question})

    = form_for(@question, url: questions_path) do |f|

        =render partial: 'form', :locals => {:f => f}
        = submit_tag("Submit")


This view uses form helpers to speed up the process of creating a form. For more info on this check out [this resource][1]. They are super useful and save a lot of time from debugging later on. This form also calls a method called render. You'll notice that the actual fields of the form aren't listed here. The reason why is that they are separated out in a partial view (or simply a partial in Rails parlance). Since the new form and edit form have a lot in common, those commonalities are typed out only once, and then when each respective form needs its fields, it simply renders the partial `_form.html.haml`. Here it is:

    %table
      %tr
        %th
          = f.label :title, "Title"
        %td
          = f.text_field :title
      %tr
        %th
          = f.label :body, "Your Question"
        %td
          = f.text_area :body

      %tr
        %th
          = f.label :tags, "Tags"
        %td
          = f.text_field :tags

      %tr
        %th
        %td
          = f.hidden_field :exercise_id, :value => @question.exercise_id 

When the user submits the new form, they are routed to the `create` controller. All of the parameters are assigned in one go by calling the private method `safe_assign`. Why? One way to assign attributes is to type

    @question.title = params[:question][:title]
    @question.body = params[:question][:body]
    ...
    zzz


But this could require a lot of typing and doesn't follow the principal of DRY (Don't Repeat Your Self). The other option would be to simply assign all of the attributes in one statement like so: `@question.assign_attributes(params[:question])` But this has serious security flaws. A bad guy could easily alter the html form and add inject additional parameters into the form. What if, for example they alter the user_id of the question? Bad news. Instead we use `require` and `permit` to specify the parameters we are letting in (this is called white listing). Every other parameter is filtered out.

## edit/update

The process to edit a question follows a very similar pattern as creating a new question:

* The user requests `/questions/:id/edit`
* The question object is looked up in the database.
* the form is populated with the current attributes (we get this feature for free with form helpers)
* The user submits their question.
* The `create` action assigns parameters and saves them.

## Error handling

What if a user makes a mistake in submitting a question? What if for example, they don't provide a title or a question body? We could make this a requirement at the database layer, but then we would get a big ugly MySQL error that our application wouldn't be able to handle gracefully. Instead we can add validations to the Question model:

    class Question < ActiveRecord::Base

        #VALIDATIONS
        validates :title,
            presence: { message: "Question does not have a title."}

        validates :body,
            presence: { message: "Question does not have a body."}

    end

Basically what's happening here is when we go to `save` a question, the object runs its validations. If the `title` or `body` is missing it will not save anything to the database and error messages will be generated. The application won't crash however, its our job as programmers to report those errors back to the user so they can correct them. I created a second partial for doing just that:

    .container
      .row
        .col-xs-12.col-md-4.col-md-offset-4
          - if record && record.errors.size > 0
            #errorExplanation
              %h2.alert-danger
                = pluralize(record.errors.size, "error") 
                prohibited this record from being saved.
              %p The following problems were encountered:
              %ul
                - record.errors.messages.values.each do |msg|
                  - msg.each do |m|
                    %li
                      =m
              %ul

If an error is encountered we keep the user on the same form and show them the reasons why the record can't be saved. This error partial will actually work for any Active Record object, so we can use it throughout the forum.

##Associating with an exercise

Finally, I wanted to add a feature where a student who is stuck on an exercise can click a help button and create a new question associated with that exercise. To do that I edited the routes file from

    resources :exercises

to

    resources :exercises do
      resources :questions, :only => [:new]
    end


This creates a new possible route ` 	/gym/exercises/:exercise_id/questions/new(.:format) `. This means that when the user hits the `new` controller they may have also submitted an `exercise_id` If so we assign it to the newly instantiated question and render the form. How do we pass that `exercise_id` to the `create` action though? HTTP is inherently stateless meaning that it has no way of knowing "where the user came from" We don't want to make the user select the exercise they want to tag it with since they have implicitly indicated the exercise by clicking a help button on that exercise page. Instead of making them enter it again, we can pre populate a field in the form, and since, they really don't need to see it, we can make it invisible. So in short, if a user submits a new question and came from an exercise page, that question is tied to that exercise. If no exercise_id was passed into the parameters, the question's exercise_id will be `nil` which is exactly what we expect. 

  [1]: http://guides.rubyonrails.org/form_helpers.html
