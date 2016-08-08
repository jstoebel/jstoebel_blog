---
layout: post
title: Getting CodeWorkout set up on localhost
date: '2016-01-31T19:55:29+00:00'
permalink: getting-codeworkout-set-up-on-localhost
---
In order to be able to contribute to CodeWorkout we will need to have the code base on either our local machines or our own server along with all of the necessary dependencies and related programs.  Here is what we did to get set up. First we visit the project's GitHub page and fork the project. Once that's done, clone the project to bring it into your local machine or development server by typing `git clone git@github.com:web-cat/code-workout.git`

Now the codebase is on you machine. Next, I followed the tutorial <a href="https://gorails.com/setup/ubuntu/14.04" target="_blank">here</a> to get my environment set up. This takes about 30 minutes.</p>
<p>Next we install the required gems for the project listed in the projects Gemfile. To do this type `bundle install` </p>
<p>Now that we have everything pulled in, we are ready to set up the database. This rails project will use a SQLlite database for development. How do we know that? We can take a peek at the file config/database.yml.  Here we find specifics on how the database should be set up for each environment. We find the following letting us know that the project will use a file called db/development.sqlite3 for its database when working in development.<br />
`<br />

    development:<br />
      adapter: sqlite3<br />
      database: db/development.sqlite3<br />
      # Pool: 16 puma threads + 10 SuckerPunch workers<br />
      pool: 26<br />
      timeout: 10000<br />

</p>
<p>So how do we make this database? Simple. Just run ` rake db:create`. A whole bunch of output will march down the terminal. Once its done we'll find the database created in the directory we expected. Next we need to create the tables and other schema to match what the project expects. Run ` rake db:migrate ` to get this done. This will run all of the migrations in db/migrate bringing our database up to date with what the current code base expects.</p>
<p>Now that the database is created, let's populate the database with some example data. Looking at the projects README we should run the following:<br />

    rake db:reset
    rake db:populate

<p>This will load up the database using the data specified in db/seeds.rb.</p>
<p>Now we are ready to fire up the app. From the root directory of the application run ` rails server ` and then in your browser visit ` <a href="http://localhost:3000/" rel="nofollow">http://localhost:3000/</a>`. A local version of the app will come up. Sweet! Now we have a locally working copy of the app and the ability to see what our changes to the code base will look like.</p><br />
