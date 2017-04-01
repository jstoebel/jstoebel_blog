---
layout: post
title: "Getting React and D3 to Play Nice"
permalink: getting-react-and-d3-to-play-nice
date: 2017-04-01 10:48:20
comments: true
description: "Getting React and D3 to Play Nice"
keywords: ""
categories:

tags:

---

I'll admit it: D3 is confusing. If you Google something like "make a pie chart in d3" you get plenty of results that will work just fine. But we aren't required dig too mush into what D3 is actually _up to_ For the most part this is a testimony to D3's awesome api. Problems begin to arise when we try to use D3 inside of a React app. The central problem boils down to a power struggle over who gets to control the DOM. D3 and React are both declarative: we state what we would like to see on the screen and they figure out, behind the scenes how to make it happen. Instead of making us manipulate the DOM ourselves (yuck) we are using a well crafted api that will manipulate the DOM based on our instructions.

This was the problem I ran into when trying to make a pie chart in my [voting app](https://github.com/jstoebel/voting) for Free Code Camp. Simply dropping a d3 pie chart from a google search into your React component isn't the right way to go because it breaks the whole React philosophy of describing what you want to see in a render method in terms of props and state. I found an impressive project [react-faux-dom](https://github.com/Olical/react-faux-dom) which creates a faux mount point for your D3 _inside of_ a React render. This was a very viable lead, but they have not yet sorted out some of many the complexities. Mostly thought, it just seemed like too much overhead for my simple use case.

At its core, a D3 pie chart is some svg elements organized together with some d3 functions. We don't _need_ to manipulate DOM nodes in the traditional D3 way like `select_all` and `append`. Fortunately I found another blog that [broke it down](https://swizec.com/blog/how-to-make-a-piechart-using-react-and-d3/swizec/6785) for me.  To build a pie chart with a legend I need the following:

 1. pie chart: an `svg` tag containing:
   - a `PieChart` component
    - props received:
      - x and y position
      - outer an inner radius size
      - data to render structured like `[{value: 5, label: 'some option'}, {}, ...]`
      - a colors function to fill each item of the pie chart based on its index
    - contents of `PieChart`:
      - a `g` element translated in terms of `x` and `y` props. contents:
        - an `Arc` component rendering each slice of the pie chart. contents:
          - a `path` element for each item of the data passed into `PieChart`. Here we are using the d3 function `d3.svg.arc()` to compute the arc.

 2. a `Legend` component:
    - props:
      - the names of the items in the pie chart (must be in same order!)
      - the same colors function passed into `PieChart`
    - contents:
      - a `ul` element, and for each name passed in:
        - a `li` element to label the name. the colors function will ensure coordination with the pie chart.

Whew! This was significantly more work than the typical D3 script that sets up a pie chart in a couple dozen lines. The benefit here is that the chart is broken down by its components. React remains in the driver's seat and everything is happy.

Yay!
