---
layout: post
title: When a small fix becomes a big issue
date: '2016-03-10T19:22:59+00:00'
permalink: when-a-small-fix-becomes-a-big-issue
---
Aside from [CodeWorkout][1] my other big project is the Education Studies [Dashboard][2] I am developing for Berea College's Teacher Preparation Program.  As a super quick summary, this project is almost a year in the making and is our attempt to build a system that is more useful and more responsive to the data and reporting needs of our department. I have found the commercial offerings in this sector underwhelming (and many of our peers agree), so last June I set out to create something better.

One import thing that happens in our program each semester is that students apply to our program. This works (roughly) as follows:

 - A student submits an application
 - A committee reviews the application and renders a decision.
 - The student receives a formal reply in the form of an admission (or denial) letter.

The app is responsible for collecting application data (who applied, when, for what certification program, etc) and also storing a the letter (the actual file) that was written for the student. Unfortunately, the initial design was not ideal based on future realities and was suffering from some technical debt. Here was the original schema:

    create_table "adm_tep", force: true do |t|
        #some attributes
        t.string   "letter_file_name",      limit: 100
        t.string   "letter_content_type",   limit: 500
        t.integer  "letter_file_size"
        t.datetime "letter_updated_at"
    end


Each application to the program has an attached document using the gem Paperclip. This is not an awful situation. But what happens if we need to attach several documents to one application? And further more, an application and a document are actually different enough entities that they deserve separate tables/models.  Why not just attach a document to a `StudentFile` record (this model already existed by the way) and simply have the AdmTep record reference it. Here's the new schema:

    create_table "adm_tep", force: true do |t|
        #some attributes
        t.integer  "student_file_id"
    end


Simple enough change right? Write the migration, rewrite the controller actions, patch the views and fix the tests? Not so fast. Things like this can sometimes feel almost like a re-factoring. Hey its achieving the same problem but just in a _slightly_ in a different way right? How hard can it be? Ok, if you're laughing right now, just remember that hindsight is 20/20 ok? What seemed at first like a simple fix ended up taking 2 days and stretching across dozens of files. Multiple controllers, their models and respective tests and a few other seemingly unrelated things were effected by that single change.

And what do you think is bound to happen when you go diving into so many areas of the code base? More re-factoring! [This talk by Katrina Owens][3] sums it up perfectly. The TL;DR version is, once you lay eyes on stale, rotting code (even/especially if its your own) you have the choice to pretend like nothing was there and whistle along your marry way. Or you can do something about it. The moral of the story: be a decent human and do something about it. 

  [1]: https://github.com/web-cat/code-workout
  [2]: https://bitbucket.org/stoebelj/eds_dashboard
  [3]: https://www.youtube.com/embed/QAUHYzC9kFM
