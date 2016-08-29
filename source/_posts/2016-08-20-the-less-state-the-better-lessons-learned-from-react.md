---
layout: post
title: "The Less State the Better: Lessons Learned from React"
permalink: the-less-state-the-better-lessons-learned-from-react
date: 2016-08-20 10:53:59
comments: true
description: "The Less State the Better: Lessons Learned from React"
keywords: ""
categories:

tags:

---
Earlier this summer I started learning React.js by working on projects through [Freecodecamp](https://www.freecodecamp.com). There was a bit of a steep learning curve at first when I first got started. If jQuery is a hammer and a bag of nails, React is a set of power tools. Ok construction metaphors are not my strong suit, the point is that while jQuery can _technically_ get the job done, scaling things up can get complicated fast. React is a more complex tool because it is made to address more complex problems.

One common problem in front end development is how to manage state, or in other words, how our code can anticipate the state of one part of the app to determine how to the app should be displayed. In a typical web 1.0 application, state is managed almost entirely in the backend (the database). The application does all of its state related reasoning in the backend and then sends static resources out to the client whose behavior is very easy to predict. jQuery might be used in some parts of the page, but they aren't expected to have much effect on the site at large (clicking a button might display a dropdown box but the rest of the page is unaffected.)  One pain point of typical jQuery based apps however is when elements begin to have an increasingly interdependent relationship with one another. Things can get confusing fast when we need to write code that needs to anticipate all of the different combinations of state inside the app.

## The ideal way

Its far better maintain the entire apps state in one central location, separate from the DOM. A database is often the best place for this, though of course that means that any change in the state of the app on the front end requires you make a request over the wire to make sure this change is saved. Using a client base store like [store.js](https://github.com/marcuswestin/store.js/) lets you save any data client side just like you would any javascript object. Depending on the problem you are trying to solve, one or both of these tools might be the right fit.

# A compromise: parent and child components

My final project in the React section of Freecodecamp was to create a [Rogue-like Dungeon Crawler game](http://codepen.io/jstoebel/pen/zBXmaL) using only Codepen (no database). The game is composed of a single `Game` component with each cell in the dungeon as a child component. By imposing the design restriction that inter-component communication would only be between parent and children I simply kept the entire game's state in the `Game` component. This means that the Game's state contains an array of objects, each one representing the state of each cell (what the cell contains and its attributes such as hit points, level etc.). This was possible because I am certain that only the `Game` component may change any of the `Cell` components. If cells needed the ability to change each other (say for example monsters need the ability to move around or fight one another) things would get hairy very quickly. Would it have been easier to do with a database? Sure! For one thing, I wouldn't have to write my own functionality to keep state and DOM elements connected (there is a lot of code to deal with this!), instead each element would be given a unique id, and when rendering would hit the database to find out what its state should be. So, lesson learned: the less state the better.
