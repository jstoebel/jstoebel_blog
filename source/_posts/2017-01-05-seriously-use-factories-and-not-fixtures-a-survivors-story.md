---
layout: post
title: "Seriously, Use Factories and Not Fixtures: A Survivor's Story"
permalink: seriously-use-factories-and-not-fixtures-a-survivors-story
date: 2017-01-05 14:54:20
comments: true
description: "Seriously, Use Factories and Not Fixtures: A Survivor's Story"
keywords: ""
categories:

tags:
 - ruby
 - rails
 -factory_girl

---

The Internet is abound with opinions on using fixtures or Factories in Rails. Being a relatively new developer, I'm usually careful about having strong opinions, but now that I've lived a one year struggle with fixtures and emerged on other side 99% fixture free in my work app, I am coming down firmly on the side of `Factory Girl`. There are clearly many smart developers out there using fixtures. I won't claim to be smarter than them, but given my experience with them, factories just make more sense for my use case. Here's my story.

## The original decision

My current work project, the EDS Dashboard at Berea College, was my first non-trivial Rails project. When the time came to select technologies for testing, one of my top priorities was to go with something that was tried and tested. Something that people smarter than me have struggled with, blogged about, asked and answered questions about on Stackoverflow. Something even a little boring. To that end I went with the advice to go with Rails default choice, `TestUnit` and fixtures. It turned out this advice may have been arguably a bit outdated as `Rspec` has seen great adoption over the years, but that's another story. (God I wish I could have my tests in `Rspec`, I just love it so much).

On its face, fixtures seems like a simple, elegant concept. We define a yaml file for each table in the app. We create records that we want inserted into the database for each test by just defining them in the file. There's easy support for foreign keys, erb tags are allowed, we can omit attributes and they are filled in for us. Easy.

The problems came as our app began to grow. We found ourselves with an overwhelming number of combinations of state that we needed to spin up and test. For example a student in the dashboard can be:

 - prospective with no forms of intention
 - prospective WITH forms of intention
 - admitted to a program
 - admitted to two programs
 - exited as a completer
 - exited by dropping out
 - ...

You get the idea. We had to constantly pull down a record from the database, modify it, add associated records, etc. Essentially fixtures only got us part of the way there.  Yes, we could have added more fixtures, but that quickly seemed like a daunting task. We were also running into problems where we had to account for preexisting state at the beginning of the test. Essentially we were starting all of our tests with baggage that had to be accounted for. I can't tell you how many times we struggled to understand why a test case wasn't passing only to discover later that "oh yeah, there's already a record from fixtures there!"

## Making the switch

Switching a developed app from fixtures to factories was not a simple process. Everyone on our team devoted time on and off over the last few months. It started with deleting all of our fixtures except for a very small number of canonical fixtures (records that are so essential to the logic of the app, that we are willing to keep them baked in to every test.) Many of the test cases were fixed with a simple one line change. Other's less so. It turns out that baked in state is like a complex series of threads. If I pull one, suddenly three others come loose too. It was a time consuming, sometimes exhausting project, but the app is better off for it. Technical debt has been paid down. The most immediate reward is that team members no longer need to account for nearly as much state when they begin writing new tests. We expect see the reward in the form of time saved start to accrue soon.

## Conclusion

Are fixtures _ever_ a better choice? Sure! But having struggled with the consequences of making the wrong choice and then struggled more with paying down that technical debt I think it comes down to a few things to consider.

 - How simple are your models?
 - How interconnected are they?
 - How many common flavors of each entity (a prospective student, an admitted student, a completer etc) are there?
 - And of course scalability. What's the likelihood any of the above will get more complex in the future?
