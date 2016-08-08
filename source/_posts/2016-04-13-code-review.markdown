---
layout: post
title: Code Review
date: '2016-04-13T23:23:15+00:00'
permalink: code-review
---
This week we are beginning to share our work with each other on Github and give feedback. Here's a summary of my insights

John did some nice work on the Questions view. He cleaned up the pages with some bootstrap and made use of the provided breadcrumb partial for our forum. The partial, it seems to me was built originally for use with the workouts portion of site. It was meant tell the user where they are in terms of the organization, workout and exercise. We had a nice discussion about the best way to generalize this partial a bit to also apply to the exercise and also be applicable to any other new portions of the site that might come along.

We also reviewed a new modal that allows the user to interact with a question. The user can response to the question, up and down vote it or report it for abuse. It looks like we would like for these interactions to be made using ajax, meaning that each of the relevant controller actions will need to response via ajax and in turn render a json response which in turn either confirms the change or renders the error message. We'll need to investigate that but in the mean time, the view looks great! Onward!
