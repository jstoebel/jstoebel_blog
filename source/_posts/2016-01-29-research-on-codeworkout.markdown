---
layout: post
title: Research on CodeWorkout
date: '2016-01-29T20:04:24+00:00'
permalink: research-on-codeworkout
---
The following is the summary of my initial research on CodeWorkout. Co-written with Austin McGlothlin.

####What is the purpose of the project?

Codeworkout is an interactive web based platform for students to practice their programming skills. Users are given programming challenges in Python, Ruby or Java and the program is able to automatically evaluate their solution. Teachers and professors can integrate CodeWorkout into their class, or students can work independently.

####How many people are working on the project?

Three contributors are listed on the GitHub page. Four people in this class will be contributing, but there have been only two active contributors since August 2015.

####How active is the project?

In the latest cycle of commits (since August 2015) there appears to have been 138 commits to the master branch made by two separate contributors.

####Where would you start if you were interested in helping with the project?

The lead for this project suggested the following ideas for contribution:

- New Content: creating/reviewing new questions.
- UI design: creating mockups in HTML for things like visualizing progression, user status, badging etc.
- Feature implementation: They have ideas for new features including a Q&A section like Stackoverflow.

Additionally, there are 8 unstaged tickets on Github tagged as enhancements and four unstaged tickets tagged as bugs.

Since Matt has made contact with the project’s lead, reaching out to him directly might be a good first step to help direct what direction we want to work in

####What usable technical documents that describe the project (e.g., functional requirements, design documents, installation documentation, etc.) exist?
- The Readme.rdoc provides some basic directions on setting up a development environment on your local machine. It also gives instructions on how to preserve, update, or populate the database for the website, including a list of what should currently be in the database to check for errors.
- Notes.txt provides more information like best practices for using git in this project, how to document work and formatting guidelines for Ruby. It’s essentially the same information as in the Readme.
- ERD.pdf is an ERD of the project generated directly from the project’s schema. This is a fantastic resource for people trying to get antiquated with the database and is so much more readable than a schema file, especially for newcomers.
- Finally the project's Gemfile is a great place to start when researching the third party gems used in this project. This will be great to reference when analyzing the code for the project to understand what’s being used and how.

 
####How are bugs and feature requests tracked?

Bugs and feature requests are tracked using the Github issue tracker.

####Are there a lot of open items?

There are not currently very many open tickets, especially considering that this project appears to be incomplete (as evidenced by the fact that users are not able to even log into the site).

####Are they being worked on? / Are issues being addressed?

All but two issues are assigned to someone in the project. Schiming over the open tickets, many appear to have some activity in the past 2 or 3 weeks.

####How long do you think it would take you to download and install this product?

Looking at the readme, it looks like getting this program running would involve the following general steps:

- cloning the project onto carter or a local machine
- setting up the environment:
    * Ruby virtual environment (RVM or rbenv)
        rails
     * installing mysql, creating a database and account with permissions
      * bundle installing all of the required gems
    * populating the database with sample data.
* Serving the app on Webbrick, rails’ lightweight web server (running ‘rails server’ from the command line.)

According to this website [https://gorails.com/setup/ubuntu/14.04][1] all of this setup would take approxiamtly 30 minutes, though in prior experience there can often be unexpected pitfalls in enviornment set-up.

####How do the developers communicate with each other? What channels or tools are available within the project?

Developers seem to communicate using the ticket tracking system in Github. Matt has also been in contact with the lead developer so that is another potential channel for communication especially as we are getting started. We may need to email the lead developer and figure out the best form of communication for them.

####What has been the most interesting thing you learned through this exercise?

 
Taking some time to research this project, we were surprised by how few issues are in Github relative to how much this project still has to accomplish. We weren’t entirely sure why this is, but one possible explanation could be that the team is trying to focus their work on what they deem the most important features and bugs and will implement other features later.

We were also surprised by how many resource files were available in the overall package that could help us out in the future, including a messy, but well thought out ER Model for the database and helpful readme files that were written out well enough to understand how we need to handle the database while modifying the website.


  [1]: https://gorails.com/setup/ubuntu/14.04
