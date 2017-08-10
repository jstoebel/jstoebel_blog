---
layout: post
title: "Upgrading the EDS Dashobard to Rails 5"
permalink: upgrading-the-eds-dashobard-to-rails-5
date: 2017-04-18 12:29:08
comments: true
description: "Upgrading the EDS Dashobard to Rails 5"
keywords: ""
categories:

tags:

---

Rails has dropped active support for Rails 4.2 which means its time to bite the bullet and upgrade the [EDS Dashboard](https://github.com/jstoebel/eds_dashboard) from 4.2 to 5. Here's a summary of some of the sticky points in that process:

# Dependency tidying

As expected, one top level dependency (`sass-rails`) was on old version and not allowing `active_record == 5.0.0`. Fortunately it is well supported and has a Rails 5 version. Easy. In this process we also identified a few dependencies we aren't using anywhere. Goodbye to `seed_dump`, `composite_primary_keys`, `active_record-acts_as`

# `rails app:update`

This script was a big help in changing the config files. One big gotcha is to take note of places where rails might want to overwrite your custom configurations. In some cases, I did some copy/pasting rather than letting rails overwrite my files.

# Dropping `dbi` gem

When trying to load the console I got [this problem](http://stackoverflow.com/questions/43454892/cant-run-rails-dbmigrate-after-upgrading-to-rails-5-0?noredirect=1#comment73969782_43454892). The problem here is the `deprecated` gem which, ironically is long abandoned and will ship support for Rails 5 on the first of Never. `deprecated` is a dependency of `dbi`. `dbi`, which has also given me some grief, is the current strategy for connecting to Banner, the enterprise database at Berea. We need a better way to connect to Banner. Fortunately, [activerecord-oracle_enhanced-adapter](https://rubygems.org/gems/activerecord-oracle_enhanced-adapter/versions/1.6.7), is well supported and lets us connect using `active_record`! Sweet! Here's how we did it:

add the following  to `database.yml`


{% highlight ruby %}
    banner:
      adapter: oracle_enhanced
      username: "me"
      password: "my_pass"
      url: <banner url> # should start with a //
{% endhighlight %}

Create an ActiveRecord model and tell it how to connect. Also, specify the table name

{% highlight ruby %}
    # models/banner.rb

    class Banner < ApplicationRecord
      establish_connection :banner
      self.table_name = <banner_table_name>

    end
{% endhighlight %}

Currently, we have a rake task that connects to banner and pulls in student data. We need to refactor that to use our new connection strategy. Currently, fetching data is handled by a connection class:

{% highlight ruby %}
    # the old way:
    class BannerConnection

      def initialize term
        #term(int) the banner term to search for.
        @term = term
      end

      def get_results
        DBI.connect("DBI:OCI8:bannerdbrh.berea.edu:1521/rhprod", SECRET["BANNER_UN"], SECRET["BANNER_PW"]) do |dbh|
          sql = "SELECT * FROM saturn.szvedsd WHERE SZVEDSD_FILTER_TERM=#{@term}"
          return dbh.select_all(sql)
        end
      end

    end
{% endhighlight %}

The object takes a term to filter by and can run a query for the filter term.. Instead, we will use a scope on our shiny new AR model to do basically the same thing:

{% highlight ruby %}
    # models/banner.rb

    class Banner < ApplicationRecord
      establish_connection :banner
      self.table_name = <banner_table_name>
      scope :by_term, ->(term) {where :SZVEDSD_FILTER_TERM => term}
{% endhighlight %}

Finally we need to:
 - ensure the `database.yml` file in production is updated.
 - that `Banner` is not included in the rails_admin initializer (`config/initializers/rails_admin.rb`)

# Updates to tests

 - using `assigns` in controllers has been moved to its own gem: `rails-controller-testing`. Its also worth noting here that Rails 5 has [changed the way attributes from controllers are parsed](https://github.com/rails/rails-controller-testing/issues/33). This broke some of my tests.
 - positional arguments in functional tests aren't allowed in rails 5.1. Instead they are passed as hash with key `params` We should clean them up now so the upgrade to 5.1 will be easier.
 - similarly `assert_equal` shouldn't be used to assert something is `nil`. I had to refactor some tests in anticipation of this breaking change in minitest 6.
 - Rails 5 is cracking down on blind params even more. Apparently tests fail when you try to use them. Glad they caught a few of them!
