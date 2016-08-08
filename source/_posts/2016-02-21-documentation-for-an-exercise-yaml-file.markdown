---
layout: post
title: Documentation for an exercise YAML file
date: '2016-02-21T15:46:03+00:00'
permalink: documentation-for-an-exercise-yaml-file
---
After speaking with Steve Edwards, we have some more clarification on the format of a exercise YAML file. Here is an example YAML file along with some explanation of what is going on.

- name (string): a common sense name to reference this exercise.
- external_id (string): id to refer to this exercise externally
  - is_public (bool): if this exercise is viewable to students
  - experience (integer): how many experience points are gained by completing 
  - language_list (string): Java
  - style_list (string): the type of question being asked (example: multiple choice, single answer)
  - tag_list (string): topics that this exercise relates to (example: initialization, objects)
  - current_version (int): the related exercise_version record
    - version: 1 
    - creator: edwards@cs.vt.edu
    - prompts: all of the prompts that go with this exercise (there can be many)
        - multiple_choice_prompt: 
            - position (int): 
            - question (string): the text of the question being asked
            - allow_multiple (bool): if multiple answers are allowed
            - choices: a listing of all multiple choice answers related to this prompt
                 - answer (string): 
                 - position (int)
                 - value (int): how many points is this choice worth
        - coding_prompt:
              - position (int):
              - question (string):
              - class_name(string): The name of the class wrapping the response
              method_name (string): The name of the method wrapping the response
              - starter_code (string): the code given to the user
              - wrapper_code (string): TBD
              - tests (string): test cases to use to test this answer. Example: 
                
                "1, 0",1,example # given as a an example before code is submitted

                "2, 4",16,example

                "3, 2",9,example

                "3, 0",1 

                "0, 3",0

                "-2, 2",4

                "2, 7",128

                "10, 3",1000

                "5, 3",125

                "3, 4",81,hidden #this test is run but not shown to the user
