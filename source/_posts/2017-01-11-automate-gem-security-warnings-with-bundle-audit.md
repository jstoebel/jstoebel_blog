---
layout: post
title: "Automate Gem Security Warnings With Bundle Audit"
permalink: automate-gem-security-warnings-with-bundle-audit
date: 2017-01-11 15:11:30
comments: true
description: "Automate Gem Security Warnings With Bundle Audit"
keywords: ""
categories:

tags:
 - ruby
 - rails

---

The EDS Dashboard is getting bigger all the time, and one concern we've had is staying on top of any  security holes in any of the gems we use. Rails is a huge community of course, and when a security patched is released, we can be sure we'll find out about it. But what about gems with smaller communities? It turns out there is a gem  called `bundle-audit` that scans your `Gemfile.lock` and consults a database to see if you've got anything that needs a patch. To get it just `gem install bundle audit` and then `bundle-audit`. You'll get a print out of any unpatched gems or gems that are grabbed from an unencrypted, http source. Sweet.

But what if we want to automate this process. I'd like to run such a check during my CI and have it fail if any problems are found. Similarly, I'd like to check my repo on a nightly basis.
`bundle-audit` can do that too!

{% highlight ruby %}

require 'bundler/audit/scanner'
include Bundler::Audit
namespace :bundler do
  desc 'Updates the ruby-advisory-db and runs audit'
  task :audit => :environment do
    scanner = Bundler::Audit::Scanner.new
    vulnerabilities = 0
    insecure_source = []
    unpatched_gems = []
    scanner.scan do |result|
      vulnerabilities += 1

      case result
      when Scanner::InsecureSource
        insecure_source << result.source
      when Scanner::UnpatchedGem
        unpatched_gems << result.gem
      end
    end # scanner

    if vulnerabilities > 0
      email = AdminMailer.gem_security_alert(unpatched_gems.uniq, insecure_source.uniq)
      email.deliver_now
      puts "#{vulnerabilities} vulnerabilities found!"
      puts unpatched_gems
      puts insecure_source
      fail "#{vulnerabilities} vulnerabilities found!"
    else
      puts "no vulnerabilities found"
    end

  end # task

end

{% endhighlight %}

If any problems are found, the rake task fails, breaking the CI and the production server sends me an alert. Easy security win.
