---
layout: post
title: "Honesty, Kindness and Inspiration: Pick Three"
permalink: honesty-kindness-and-inspiration-pick-three
date: 2017-05-23 08:16:56
comments: true
description: "Honesty, Kindness and Inspiration: Pick Three"
keywords: ""
categories:

tags:

---

Folks, we need to talk about a tweet:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Code review can be:<br><br>• honest<br>• nice<br><br>Pick one...</p>&mdash; I Am Devloper (@iamdevloper) <a href="https://twitter.com/iamdevloper/status/864410644732313600">May 16, 2017</a></blockquote>

As far as I can tell, this seems to be the attitude among many developers: I can either 1) give you feedback that is touchy feely and makes you feel good but is useless or 2) We can get some **real** work done but I will make you feel like garbage. The Tweet received several objections from people wondering aloud why feedback can't be both honest **and** kind.

In a past life, I was a high school drama teacher. Balancing critical feedback with kindness was something always at the forefront of my mind. One common approach was the so-called "Shit Sandwich", (and no, I never called it this on the job), where you pad negative feedback in between positive feedback. An over simplified summary of how it works would be something like this: "I really liked how you did ________, but that part where you did _________, was not so good. You should do _________ instead. ...But on the whole it was great!"

This felt a little better to me, because it helped manage the sometimes delicate feelings of teenagers. The problem was that the receiver of the feedback sometimes thought of it like a report card: "I got two good grades and one bad one. On the whole I've got a decent GPA." The creator thought of the good spots as as uniformly good, not needing change and the bad spots as unfixable, impossible to get any better at. **Shouldn't the whole point of feedback be to inspire the creator(s) to go back to their work with fresh eyes, excited about making their creation even better?**

### Liz Lermon Critical Response Process

Liz Lermon a dancer, choreographer and founder of the Liz Lermon Dance Exchange developed a framework for feedback called the [Critical Response Process](http://danceexchange.org/projects/critical-response-process/). It has been used by artists, administrators, scientists, academics, and even in the corporate sector. It isn't constrained to works that we would traditionally call "Art". Really, its for anything creative. Software is a perfect use case for this! Here's how it works. The creator(s) sit down with a group of responders. The goal is for the creators to see a way forward for improvement. One person acts as the facilitator. The feedback process follows four steps:

#### Step 1: Statements of Meaning

Responders state what about the work had meaning, was interesting or stood out. We can think of this as a way to generate material of what to talk about in more depth later in the process. The important part here is that **statements are not evaluative in this step.** We save those for later.

Some good examples of statements of meaning:

-   I noticed that you're not using Webpack with this project.
-   I noticed that the code is written in a functional style.
-   I noticed that the codebase has no dependencies outside the standard library.

Some bad examples of statements of meaning:

-   This code is using Python 2. All new Python projects should be using Python 3 at this point.
-   This code is vulnerable to SQL injection. You need to fix it.

To be clear: this process isn't trying to somehow silence concerns of security, performance or anything else important. It only asks that those comments be saved for later.

#### Step 2: Questions from the Creator(s)

Next, the creator(s) ask the responders questions about their work. Responders may give their opinion about things explicitly asked about.

A good example:

-   Q: In this part of the documentation I tried to make it clear what this method does and why. Was it clear to you? And if not, how could I make it more clear?
-   A: I think I understand how its used but it wasn't clear to me what its purpose was. Maybe think about including an example use case.

A bad example to the same question above:

-   A: No, and in general the documentation was hard to follow and not so well organized. The way you should organize it is...

In the second example, the responder's comments were out of scope of what the creator asked about. Here the responders should answer the questions and nothing more. Opinions come at the end.

#### Step 3: Neutral Questions from the Responders

The idea here is that feedback is best when the responder has the right context. Here, responders ask questions that do not have an embedded opinion.

An example of a neutral question: What ideas guided you to select Python 2 for this project?

An example of a question with an embedded opinion: Why are you using an old version of Python?

It doesn't matter that the phrase "an old version of Python" is an objective one. By using the word "old" the responder is making their opinion clear: "you shouldn't be using Python 2". Again, **opinions are fine, but they come at the end**. Which brings us to...

#### Step 4: Opinions

Here's where responders may give their opinions on the work with the consent of the creator(s). It works like this:

Responder: I have an opinion about response times in production. Would you like to hear it?

Creator(s): Sure!

Responder: I can see a situation where this bit could be computationally expensive and could lock up the server for a while. You might consider handing it off to a different process.

Or...

Responder: I have an opinion about the use of Python 2 in this project, would you like to hear it?

Creator(s): No thanks!

I realize that the idea of express consent before all opinions might not work in all situations. In a traditional corporate structure, a team doesn't get to just say "no thanks" to their manager's opinion. Here, consent for opinions (so long as they are thoughtful and well timed) is implied by the relationship. The manager can simply give her opinions at this point without needing to ask if they are allowed. The politics of this are entirely dependent on your organizational culture. You'll need to figure out what's right for your team.

And that's pretty much it. Again, the idea is for the creator of the work to receive thoughtful, critical, **useful** feedback. Once the session is done, the creator(s) should feel inspired to go back and make their work even better.
