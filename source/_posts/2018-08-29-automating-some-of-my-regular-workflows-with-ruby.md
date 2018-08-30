---
layout: post
title: "Automating Some Of My Regular Workflows With Ruby"
permalink: automating-some-of-my-regular-workflows-with-ruby
date: 2018-08-29 16:42:01
comments: true
description: "Automating Some Of My Regular Workflows With Ruby"
keywords: ""
categories:

tags:

---

At my day job, I maintain dozens of different rails projects for various clients. One thing I need do to frequently throughout my day is deploy projects, both to staging and production. We use capistrano to handle all of our deployments, but sometimes just the command to kick things off can be too much typing/thinking. For example this is typically the command I use to deploy a project to staging

```
BRANCH='some_branch' bundle exec cap staging deploy:clobber_npm deploy:clobber_lock deploy;
```

After doing some thinking I realized, _this is a lot of typing for what amounts to the same thing over and over_. In most cases I want to do one of two things.

1) Deploy to stage, my current branch, while clobbering the lock and npm
2) Deploy master to production

Going with the the rails philosophy of embracing sensibile defaults, I created a CLI command that let's me perform the most typical things by default and have control when I need to configure something.

To deploy my current branch to stage its just:

```
epub deploy
```

to deploy to production:

```
epub deploy --stage="production"
```

And when I need to, I can control the branch and if I am clobbering the deploy lock and/or npm

```
epub deploy --stage="production" --branch="my_other_branch" --clobber_npm --clobber_lock
```

## Set it an forget it

On some days I deploy up to a dozen times or more. Often, more than one project is working at a time. I can sometimes get overwhelmed by all of the tasks I'm trying to juggle in my mind at one time. That's why I have an audio message set to let me know when a command is finished. That way I can put it out of my mind and know that the machine will let me know when things are done and I can take more action. In the mean time I can get some tea or work on another task.

## Coping Databases

An even more painful workflow that I was experiencing was copying a database from staging or production to my local machine. To do that I had to: 

 1. ssh into the app server
 2. run `rake db:backup`
 3. Wait while the database is dumped, trying not to forget about it.
 4. `scp` the file down. Wait some more.
 5. unzip the file.
 6. run the resulting sql file against my local database. More waiting.

All of this can take 15 minutes or more. That's time spent trying to work on something else, while having to keep part of my attention on how this is going. If I wan't paying attention, I could run `rake db:backup` and then forget all about it. Way too much context switching! We used to have a command in all of our projects call `rake db:copy` which pulled down the database all in one step, but its implementation was found to be problematic for production so we had to scrap it. I wanted to write a script that set out to automate all of the steps that I normally perform one by one, but not requring any user input in the middle. So I made `epub copy_db`. Now I can issue the command, devote my attention to more important things and come back when everything is ready to go.
