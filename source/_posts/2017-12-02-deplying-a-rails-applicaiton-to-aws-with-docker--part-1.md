---
layout: post
title: "Deplying a Rails Applicaiton to AWS with Docker  Part 1"
permalink: deplying-a-rails-applicaiton-to-aws-with-docker--part-1
date: 2017-12-02 12:55:08
comments: true
description: "Deplying a Rails Applicaiton to AWS with Docker- Part 1"
keywords: ""
categories:

tags:

---


# setting up dev enviornment

 - followed this tutorial: https://docs.docker.com/compose/rails/
 - need to mount my host system to get at source code:
 - needed to persist postgres data back to host system once container is finished.
 - rake task to start up
 - need to point at the `docker-compose.yml` and give correct build context

I've been working with Docker and Rails for a bit now and really love it. It makes for huge wins both in terms of setting up your development environment and deploying to AWS, but the steps to get started aren't always obvious. That's why I decided to document my steps in a series of blog articles. In this first part, I'll go over how to set up a development environment.

# My assumptions
 - you have a basic familiarity with Rails and how to set up a project
 - You know how to read YAML
 - You have Rails and Yarn installed on your local machine
 - You have heard of Docker before and at least vaguely have a sense of what it does, even if you've never used it. (If not that's ok, spend a few minutes Googling and then come back.)

# Why would I want to do this?

Recently there was something strange going on with my instillation of either Postgres or Homebrew on my Macbook. Rather than working to get the bottom of it for the umpteenth time I decided to just run my Rails project locally in a Docker container.

# Start your rails project

To keep this example as simple as possible I am going to spin up a new Rails project and then get it running inside a container. Let's stat by creating a new Rails project:

```
rails new docker_test --database=postgresql --webpack="react"
```

We get a whole bunch of output as rails generates then project and then sets up for webpack. The webpack bit isn't strictly speaking a requirement for this, but I think its helpful to remember that most real Rails projects will need to have some fronted assets built before running in production. React is my preferred front end library but really anything would do here.

Next I am going to generate a super simple scaffold, really just as a way to verify that my database is working. This will be a trivial app that lets me record the names of my friends.

```
rails g scaffold friend name:string
```

More output verifies that my scaffold files have been generated. Normally at this point my next step would be to `bundle exec rake db:migrate`, But we're not trying to use Postgres on my local machine. I want to use the postgres image inside of a Docker container.

# Set up Docker

Let's get set up to run this app inside Docker. First in your root directory create the file `Dockerfile`. Then inside that file drop the following:

```
FROM ruby:2.4

# basic dependencies
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

# Install capybara-webkit deps
RUN apt-get update \
    && apt-get install -y xvfb qt5-default libqt5webkit5-dev \
                          gstreamer1.0-plugins-base gstreamer1.0-tools gstreamer1.0-x

# Node.js
RUN curl -sL https://deb.nodesource.com/setup_7.x | bash - \
    && apt-get install -y nodejs

# selenium
RUN apt-get install -y python-pip
RUN pip install selenium

# Chrome
RUN wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
RUN dpkg -i google-chrome-stable_current_amd64.deb; apt-get -fy install

RUN mkdir /docker_test
WORKDIR /docker_test
ADD Gemfile /docker_test/Gemfile
ADD Gemfile.lock /docker_test/Gemfile.lock
RUN bundle install
ADD . /docker_test
```



Let's step through this file chunk by chunk. The first line `FROM ruby:2.4` indicates the base image that you're docker container will be staring from. In this case, your pulling in Ruby 2.4 into your container. The next several `RUN` commands through to `RUN dpkg -i google-chrome-stable_current_amd64.deb; apt-get -fy install` are installing dependencies into the container. This gets us setup for the things we will need in a realistic rails project. Things like Capybara and Selenium for integration tests. I'll have more to say about this file in a minute.

Next create a directory `config/docker/development`. In that directory create the file `docker-compose.yml`. Inside that file place the following:

```
version: '3'
services:
  db:
    image: postgres
    ports:
      - "5432"
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/docker_test
    ports:
      - "3000:3000"
    depends_on:
      - db
```

# Build container and set up project

Next we'll get the environment set up:

```
docker-compose -f config/docker/development/docker-compose.yml --project-directory . build
```

This command says: "using the `docker-compose.yml` file (the -f flag indicates its not in the current directory) please build this project. the `--project-directory` flag tells us the context of this project (in this case the root of our rails project). If we had not provided this argument, when docker goes to build my project, all relative paths would be relative to the location of the docker-compose file. Directories above that directory would not be allowed.

Why does this matter? Take a look at the `docker-compose.yml` file. For example, under `web -> volumes` we have the value `.:/docker_test`. The idea here is that we are providing the container access to the entire code repo. If I make changes to the source code while developing, I won't need to rebuild the whole thing for my container to have access. If I had not set `--project-directory` to point at my rails root, then my container would have access to the location of the docker-compose file, specifically, `config/docker/development`. Not what we want. Its also worth pointing out the line `./postgres-data:/var/lib/postgresql/data` under `db -> volumes`. This specifies that data created in the container will be persisted back to my local machine. You probably don't want to check this data into source control so be sure to add it to `.gitignore`.

Next we are ready to start up the project. `docker-compose -f config/docker/development/docker-compose.yml --project-directory . up`. You should get output similar to the following?

```
dockertest_db_1 is up-to-date
Starting dockertest_web_1 ...
Starting dockertest_web_1 ... done
Attaching to dockertest_db_1, dockertest_web_1
db_1   | 2017-12-04 18:46:45.546 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
db_1   | 2017-12-04 18:46:45.546 UTC [1] LOG:  listening on IPv6 address "::", port 5432
db_1   | 2017-12-04 18:46:45.693 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
db_1   | 2017-12-04 18:46:45.954 UTC [23] LOG:  database system was shut down at 2017-12-04 18:22:39 UTC
db_1   | 2017-12-04 18:46:46.193 UTC [1] LOG:  database system is ready to accept connections
db_1   | 2017-12-04 18:47:10.441 UTC [30] FATAL:  database "docker_test_dev" does not exist
db_1   | 2017-12-04 18:47:27.993 UTC [1] LOG:  received smart shutdown request
db_1   | 2017-12-04 18:47:28.025 UTC [1] LOG:  worker process: logical replication launcher (PID 29) exited with exit code 1
db_1   | 2017-12-04 18:47:28.028 UTC [24] LOG:  shutting down
db_1   | 2017-12-04 18:47:28.070 UTC [1] LOG:  database system is shut down
db_1   | 2017-12-04 18:48:53.186 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
db_1   | 2017-12-04 18:48:53.186 UTC [1] LOG:  listening on IPv6 address "::", port 5432
db_1   | 2017-12-04 18:48:53.191 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
db_1   | 2017-12-04 18:48:53.261 UTC [23] LOG:  database system was shut down at 2017-12-04 18:47:28 UTC
db_1   | 2017-12-04 18:48:53.357 UTC [1] LOG:  database system is ready to accept connections
web_1  | => Booting Puma
web_1  | => Rails 5.1.4 application starting in development
web_1  | => Run `rails server -h` for more startup options
web_1  | Puma starting in single mode...
web_1  | * Version 3.11.0 (ruby 2.4.2-p198), codename: Love Song
web_1  | * Min threads: 5, max threads: 5
web_1  | * Environment: development
web_1  | * Listening on tcp://0.0.0.0:3000
web_1  | Use Ctrl-C to stop
```

Our project is now running! You should be able to access it from `localhost:3000`. 

Next we need to set up the database and run migrations _inside_ the container. To do this we have a command called `run`. In a separate console tab run:

```
docker-compose -f config/docker/development/docker-compose.yml --project-directory . run web rake db:create db:migrate
```

This tells your service labeled `web` to run the given rake commands. Sweet, our project is up and running!

# Using rake to do less typing

Typing out `docker-compose -f blah blah blah` is tiersome! Let's make a rake task to make it easier:

```
desc 'ensure no sneaky rails envs.'
task :assert_env do
  raise 'Invalid env' unless /production|development/ =~ Rails.env
end

desc 'Run docker-compose command with the correct settings '
task docker_compose: %w[environment assert_env] do
  options = {}
  OptionParser.new do |opts|
    opts.banner = 'Usage: rake docker_compose -cmd'
    opts.on('-c', '--command ARG', String) { |cmd| options[:cmd] = cmd }
  end.parse!

  command_str = 'docker-compose ' \
      "-f config/docker/#{Rails.env}/docker-compose.yml " \
      "--project-directory #{Rails.root} " \
      "#{options[:cmd]}"

  puts command_str
  exec command_str
end
```

The rake task `docker_compose` does the following:

- asserts that `RAILS_ENV` is either `development` or `production`. We could add other's later, or remove this assertion entirely, but since we are using `Rails.env` to reference a directory I wanted to put some constraints around acceptable inputs.
- Parses the command passed to the `-c` flag. This will be the bit that comes after the long and boring `docker-compose -f config/docker/development/docker-compose.yml --project-directory .`
- prints the command to screen
- execs the command so we can see its output.

For example, to bring my container `up` I type `bundle exec rake docker_compose -c up`. To run a rake task against it: `bundle exec rake docker_compose -c "rake db:migrate"` Yay! 

In the next few posts I'll talk about how to set up a docker for production and deployment.

