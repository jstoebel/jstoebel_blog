---
layout: post
title: "Deploying eds_dashboard with Capistrano"
permalink: deploying-eds_dashboard-with-capistrano
date: 2016-03-07 00:00:00
comments: true
description: "Deploying eds_dashboard with Capistrano"
keywords: ""
categories:

tags:
 - Ruby on Rails
 - eds_dashboard
 - Capistrano
---

Capistrano is a great way to automate deployment of new versions of a Rails app. Even with a relatively simple deployment like we have for the EDS Dashboard (one server) its becoming clear that we don't want to take the chance with human error when it comes to deployment. The more "set it and forget it" we can do the better. Capistrano has [plenty of documentation](https://www.phusionpassenger.com/library/deploy/apache/automating_app_updates/ruby/) out there but since it is so flexible, the important thing is to document how we are configuring this particular project:

## Configuration

Capistrano does its job using several files loaded into your repository:

- **Capfile** lists a series of recipes that Capistrano needs to perform. Common tasks like running `bundle install` `rake db:migrate` or restarting your web server are typically included as external libraries and they usually just work right out of the box.


 - **config/deploy.rb** Contains a series of tasks you can run for deployment. The task that comes out of the box is as follows:

 ```
 namespace :deploy do

   after :restart, :clear_cache do
     on roles(:web), in: :groups, limit: 3, wait: 10 do
      # put anything else you want here.
     end
   end

 end
 ```

 In our case this is perfect for us as is. It deploys our code, runs all of the set up, restarts the web server and we are good to go. If we needed other intermediate tasks we could put them here.

- **deploy/production.rb and deploy/staging.rb** These files provide more specific control for different environments. One thing we definitely need to provide here is the server name and user name of our production server. Capistrano will SSH into this server so be sure that SSH keys are set up properly.


There are some other settings here that are worth a quick explanation

 - `deploy_to`: the location of where the application should be deployed. If nothing is placed here, the default will be /var/www. and _not_ in the user's home directory.
 - `default_env`: any other environment variables you need set when running the deployment. In our case we need the location of our Oracle client in order to setup `ruby-oci8`.


 ## Shared Files

This topic was a bit tricky, so its worth going into. There are some files in our app that for various reasons are not checked into source control and therefore persist or are shared between each deployment. In our case, this is `database.yml`, `secrets.yml` and all of the uploaded student files in `public/student_files`. Capistrano provides a shared location for these files, conveniently called `shared`. The currently deployed version of the app will reference the files in that directory. Setting this up takes two steps:

#### specify which files and dirs will be referenced

In `deploy.rb` indicate which files we be referenced in `shared`:

```
set :linked_files, fetch(:linked_files, []).push('config/database.yml', 'config/secrets.yml')
set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system', 'public/student_files')
```

#### move the files to the shared location

In our case this means we need to make the following moves
 - `config/database.yml` -> `shared/config/database.yml`
 - `config/secrets.yml` -> `shared/config/secrets.yml`
 - `public/student_files` -> `shared/public/student_files`

These files will live there permanently. Deployments of the app will come and go, each referencing these locations in turn, but as old deployments are removed, these files won't be affected.

## A note about `Rails.root`

You may be asking your self, "self, my files have moved into `shared`. Does that mean I must change the path to reference them in any model using `Paperclip`?" The answer is no and the reason is because `Rails.root` is automatically altered to reference files in the current deployment. You can check it our for yourself if you are curious, but essentially, if you move the files to share, and specific them in `deploy.rb` the app will find them like normal. No changes to models needed!
