---
layout: post
title: Think twice before you change your Rails primary key data type
date: '2016-04-19T13:22:21+00:00'
permalink: never-change-your-rails-primary-key-data-type
---
In most cases, a database table does just fine with an auto incrementing integer as its primary key.  We are taking a piece of data "from the wild" and representing it in a database table. The first thing the record needs is a unique identifier. Not having one naturally, we just give it an arbitrary number. A blog post for example is an abstract construct. We simply assign an arbitrary, unique number and move along. This is such a common concept that ActiveRecord (the most popular Ruby ORM, and default for Rails) uses this practice by default. Here the id, defauls to auto incrementing integer implicetly. 

    class AddPosts < ActiveRecord::Migration
        def change
            create_table :posts do |t|
                t.string :title
                t.text :content
                t.timestamps
        end
    end

For my fellow Python fans who are shouting "hey! [Explicit is better than implicit][1]" calm down. This is Ruby, not Python. There's something to be learned from the wisdom of another community.

But what happens if we want our primary key to be a non integer? Here at Berea College, student ids are formatted as the string "B00" followed by six digits (example "B00123456"). Rails allows for us to set a string a primary key in the migration file by doing this:


    class AddStudents < ActiveRecord::Migration
        def change
            create_table "students", :id => false do |t|
                t.string "Bnum", :primary_key => true
                t.string  "FirstName"
                t.string  "PreferredFirst"
                t.string  "MiddleName"
                t.string  "LastName"
                t.string  "PrevLast"
                t.string  "ProgStatus""Prospective"
                t.string  "Major"

            end
        end
    end

This works as expected when we migrate, but the problem arises when we take a look at the file `schema.rb`

      create_table "students", primary_key: "Bnum", force: true do |t|
        t.string  "FirstName"
        t.string  "PreferredFirst"
        t.string  "MiddleName"
        t.string  "LastName"
        t.string  "PrevLast"
        t.string  "ProgStatus"
        t.string  "EnrollmentStatus"
      end

What gives? The rake task `db:schema:dump` completely ignored our wishes to make the primary key a string. This is a [documented issue][2] but it won't be fixed any time soon. You may ask why this is a problem. Can't we just always build the database using `db:migrate`. We _can_ do this, but that would mean that every time we deploy we need to run _all_ of the migrations to get the database to the state we want it. That means taking the twisty, turny, Candy Land type journey to get from a fresh database to where we need it. As our collection of migrations grows this can cause significant slow down in our work flow. A functioning `schema.rb` on the otherhand will take us straight to where we need, from point A directly to point B.

## workaround 1: using raw SQL

If we are sure we want to use non integers as primary keys we can do the following

 1. Open up `config/application.rb` and change `config.active_record.schema_format = :ruby` to `config.active_record.schema_format = :sql`
 2. This won't however change the behavior of `db:schema:dump` (the task that is run when ever we migrate). To fix add the following rake task to your project to dump sql after migration.

     Rake::TaskManager.class_eval do
      def remove_task(task_name)
        @tasks.delete(task_name.to_s)
      end
    end

    Rake.application.remove_task('db:schema:dump')
    namespace :db do
      namespace :schema do
        task :dump do
          Rake::Task['db:structure:dump'].execute
        end
      end
    end 

##workaround 2: refactor to use the defaults

In my limited experience, I've noticed that Rails uses sensible defaults to guide the user towards a smart design. My attempt to not use integers as a primary key is a perfect example of this. While I could go with the above change, I will instead be refactoring my database to use all auto-incrementing integers as primary keys. While this may be a bit of work in the short term, my intention is to get my project a little bit closer with the rails convention and (hopefully) avoid larger problems down the line. The morale of this whole story? Let the conventions guide your design unless you have a _really_ good, well researched reason not to.


  [1]: https://www.python.org/dev/peps/pep-0020/
  [2]: https://github.com/rails-sqlserver/activerecord-sqlserver-adapter/issues/395
