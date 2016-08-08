---
layout: post
title: Communication in a large team
date: '2016-04-10T14:22:28+00:00'
permalink: communication-in-a-large-team
---
Some of the challenges in working in a six person team are everyone staying in sync about the direction of the project. A perfect example of this occurred this past week as we were working on our Q&A forum. Before I go into details of this, I should stress that I don't believe anyone is to blame for any of this confusion. We are all still learning and I wager that working in a leaderless six person team is new to all of us. I am writing this blog post as a reflection of a learning experience and not to place blame. For this reason I won't use names in this post.

## Welcoming new team members

 We took on two new team members when the OpenMRS team had to fold a few weeks back. From the start I believe that we did not set forth clear expectations as to how our new team members should contribute to the project. Were they going to contribute to the Q&A forum or find another project within CodeWorkout? If they were going to work with us, how were we going to adjust to welcome them? We never discussed it.

## Starting version 0.1
At about that same time, our team had finished implementing version 0.0 of the Q&A forum and we were ready to start on version 0.1. We brainstormed ideas for new features and divided them up. Something we had not done, was discuss how our work would interact with each other. Discussions such as "feature X is being worked on by person A, and that interacts with person B's work. You two should plan to work together closely so your contribution can work together smoothly"

##Conflicting views

Someone on our team revised a view on our forum and made a pull request on it. I merged it in to master (note this this is the only time I am singling anyone out in this post). As it turned out, another team member had also planned to work on this view. We are now faced with the challenge of merging the work between these two team members. 

Having not communicated clearly as a team, it is perfectly understandable that this happened. 

##We can do this!

Fortunately all of this is something that we as a team can work through. As we approach the end of the semester, here are some things that I think I could do better. If any of my team members find this useful, so much the better.

 - Know what my team is working on. If I think something I am doing affects someone else's work, make it my business to work with them early on. Trying to sort out merge differences after the fact is less efficient than getting synced up early on.
 - Review before merge: The whole point of the pull request process it is create a venue for a dialogue about changes to the code base. For the sake of expedience I merged my team members PR without much review. I didn't want to hold up the process much longer. While its true that we don't have much time left, I'm afraid I inadvertently introduced more confusion to the system.
 - Pair more: I think the solution to all of this might be to pair more often. It allows us to questions each other's assumptions and catch miscommunication more frequently.
