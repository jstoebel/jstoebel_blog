---
layout: post
title: Adding an Exercise in CodeWorkout--The Empire Strikes Back
date: '2016-02-17T01:52:49+00:00'
permalink: adding-an-exercise-in-codeworkout-the-empire-strikes-back
---
####TL:DR
I still haven't figured out how to add an exercise, but I'm closer 

This is the next chapter in my saga in working out how to enter an exercise in CodeWorkout. This evening as I was digging around the app's routes file I found two encouraging paths

    exercises_import_path 	GET 	/gym/exercises_import(.:format) 	exercises#upload_yaml

    exercises_yaml_create_path 	POST 	/gym/exercises_yaml_create(.:format) 	exercises#yaml_create 

It looks like there is some infrastructure for uploading and processing a YAML file of an exercise after all. I tried typing in `/gym/exercises_import` and get an error: missing template. Looks like this hasn't been implemented yet. I went ahead and made my own, just to play with:

    #upload_yaml.html.haml
    I am upload_yaml
    </br>
    find me at gym/exercises_import
    = render :file => 'exercises/upload.html.haml'

Fortunately, a simple form to select a file is already provided in `upload.html.haml` which I render on the final line.  Here's that form:

    = form_tag exercises_url + '/exercises_yaml_create', multipart: true do
    = file_field('form', 'file')
    = submit_tag 'Submit File'

The form even seems to know where to send the request.

Now let's make a yaml file to feed in. I made a very simple .yaml file from `exercise-format.txt` 

    Title: Python

    Language: Python
    Tags:     if, condition, plus, integer


    Stem: "Stem goes here"

    Code: print

    Explanation: "Explanation here"

    Tests: "Test here"


Great! So let's try uploading the file and we get...an error: `undefined method `has_key?' for ["Title", "teenSum"]:Array` Occurring here:

    collection_representer class: Exercise, instance: lambda { |fragment, i, args|
    if fragment.has_key? 'external_id'
      e = Exercise.where(external_id: fragment['external_id']).first
      e || Exercise.new
    else


Hmmm, I'm not exactly sure what's going on here . Doing a bit of inferring it looks like this is a decorator class that is looking to see if we have included an external_id in the YAML file and if so finds and exercise with that id. Failing that, it simply creates a new Exercise. 

Let's try a different tact:  I also see there is also another action in this controller that has something to do with creating exercises: `yaml_create`. Let's try to send the file over there. Trying to do so initially throws an error. It was expecting a param called :yamlfile but it only got :file. Fixing that quick, we then get a second error on line 340 no implicit conversion of String into Integer for   `@ex.name = exercise['name']` Taking a glance at the controller I can tell what is *meant* to happen but some piece of the puzzle is not working right. Either my yaml file is not composed properly or their is something wrong with the code in the controller. I will have to keep digging.
