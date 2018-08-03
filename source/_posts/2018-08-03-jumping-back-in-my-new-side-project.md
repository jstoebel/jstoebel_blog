---
layout: post
title: "Jumping Back In: My New Side Project"
permalink: jumping-back-in-my-new-side-project
date: 2018-08-03 12:58:18
comments: true
description: "Jumping Back In: My New Side Project"
keywords: ""
categories:

tags:

---

Since my son was born last Spring, I've been pretty much heads down on 1) taking care of him with my wife and 2) trying to do well in my new job with ePublishing. There hasn't been a whole lot of time for side projects / outside learning. That will be changing though.

Thanks to an incredibly supportive wife and some creativity with my schedule, I am devoting some time to growing as a developer. Some of my goals:

 - Get some experience building a more complex project form the ground up. This means choosing the entire tech stack considering the trade offs and implementing it.
 - Level up my skills with React, TypeScript and maybe later React Native.
 - Make a project that has at least two people that will care about it (me and my wife)

Here's what I have in mind for a project. In my current job I need to keep meticulous track of my time for the purpose of billing clients. I currently use Toggl and like it ok, but I find myself wishing I could hack on it to add a small feature here or there. Since that's not really possible, why not just build my own? Even if I don't end up using it on a day to day basis, the experience of building something that actually matters to me seems like a better way to go, than just building a random side project. Some ideas for this project include:

 - Set up various projects in your life and track the time spent working on them.
 - See a summary of time spent on various projects. Compare time spent with the established goal. 
 - Keep notes on what you accomplished in a given work session
 - Run a pomodoro clock for a particular task.
 - And what ever else feels interesting to me.

# The technology

Let's be honest, part of the reason I am doing this project is an excuse to learn some new technologies. This is what I have in mind for a tech stack: the app will run for the desktop as an electron app. I'll build the front end out with React written in TypeScript and persist the data to a local database (I'm thinking just sqlite, but I may change my mind here). I am also itching to learn GraphQL, so I will hook that up as well. I will write tests for everything using using Jest and Enzyme and lint everything with eslint.


## Supported Platforms

 - Desktop (Window and Mac OS)

## Language
 - Typescript

## Front End
 - React
 - Electron
 - Babel/Webpack

## Back End
 - Sqlite
 - GraphQL

## Code Quality
 - jest / enzyme
 - eslint

# Getting ambitious

There's a lot of other places this could project could go in the long run. What if I wanted to also support mobile and web versions of the app. That would give me some more experience writing React components across multiple platforms, and gaining some experience scaling out a project. Things may catch fire here, but that's part of the learning experience!

## Supported Platforms

 - iOS and Android
 - Desktop (Window and Mac OS)
 - Web

## Languages
 - Ruby
 - Typescript

## Front End
 - React
 - React Native
 - Electron
 - Babel/Webpack

## Back End
 - Ruby on Rails
 - GraphQL
 - Postgres

## Code Quality
 - jest / enzyme
 - rubocop
 - eslint


Will any of these change? Of course! This is a learning experience!

## Next steps

I can't just start writing code on this project straight away. Here's what I need to do to get ready to start on my project.

 - Learn a little more about TypeScript and how to set it up with a project (new to me!)
 - Continue my learning with Electron and identify any gotchas for how my project will differ from running on the web.
 - Learn about how GraphQL and its eco system works (also new to me?). Plan out how I can support local first persistance should I want to store data in the cloud eventually.
 - Plan just a few items for a road map.
 - Build a hello world!