---
layout: post
title: "automating modifying dbi at each deploy"
permalink: automating-modifying-dbi-at-each-deploy
date: 2017-01-04 15:19:42
comments: true
description: "automating modifying dbi at each deploy"
keywords: ""
categories:

tags:
 - ruby
 - rails
 - capistrano

---

The [EDS Dashboard](https://github.com/jstoebel/eds_dashboard) needs to connect to Berea College's enterprise database (Banner hosted using Oracle). To do this we use [ruby-oci8](https://github.com/kubo/ruby-oci8) which uses [dbi](http://ruby-dbi.rubyforge.org/) as a dependency. `dbi` is an old library and does not technically support Ruby 2.x. Fortunately with a change of a [single line](http://stackoverflow.com/questions/27873121/dbi-row-delegate-behavior-between-ruby-1-8-7-and-2-1) the library works with our use case. I had initially solved this problem by stepping into the source code inside the production server and making the change. This worked fine until we switched to Capistrano a few months back. In the Capistrano flow, gems are pulled down and bundled at each deployment. I _could_ manually edit the source code at each deployment, but the whole purpose of Capistrano is automated deployment (script everything, minimizing room for error). We need a way to automate this.

## Update dbi?

I headed over to the [Github page for dbi](https://github.com/erikh/ruby-dbi/pull/14) and...tumbleweeds. Someone proposed the trivial changes to support Ruby 2.x more than 3 years ago and they remain unmerged.

## Use my own fork?

I could fork the project and push it up to Ruby Gems. That would be a perfect solution if this was a top level dependency. Just make the swap in the Gemfile and we are good to go. However `dbi` is a dependency of `ruby-oci8` and I have no power over the gems that they include. I _could_ fork this library as well, but then I am responsible for keeping it up to date from upstream. There could be major security flaws that go unattended. Seems like a bad idea and more work that its worth.

## Automate the hard edit

Let's step back and think about what we need to get done. All we need to do is automate the editing of a single line. Why not just ask Capistrano to execute a bash command against the appropriate file once deployment is done? Capistrano provides a dead simple way to do this using after hooks. Add this to `lib/capistrano/tasks/deploy.rake`

{% highlight ruby %}

namespace :deploy do

  desc "make changes to dbi library for compatibility with Ruby 2.x"
  task :update_dbi do
      # update file in dbi per these instructions: http://stackoverflow.com/questions/27873121/dbi-row-delegate-behavior-between-ruby-1-8-7-and-2-1
      file_loc = "/home/stoebelj/eds_dashboard/shared/bundle/ruby/2.3.0/gems/dbi-0.4.5/lib/dbi/row.rb"
      puts "modifying #{file_loc}..."
      on roles(:app) do
        # modify the source code in place
        execute(" sed -i '212s@.*@        if RUBY_VERSION =~ \/^1\.9\/ || RUBY_VERSION =~ \/^2\/ @' #{file_loc} ")
      end
  end

  after :deploy, "deploy:update_dbi"

end


{% endhighlight %}

 Essentially we just provide a rake task to be run after deployment is done. This task runs the unix command `sed` which replaces line 212 with the text `         if RUBY_VERSION =~ \/^1\.9\/ || RUBY_VERSION =~ \/^2\/`. Also, TIL sed can use what ever delimiters you want. So if you're trying to insert a regex with sed, just use any other delimiter. No more slash escape hell!
