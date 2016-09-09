---
layout: post
title: "Scales in D3"
permalink: scales-in-d3
date: 2016-09-09 05:11:25
comments: true
description: "Scales in D3"
keywords: ""
categories:

tags:

---

This week I began the D3.js section of the [Freecodecamp](https://www.freecodecamp.com/) curriculum. In the first challenge we are given an endpoint with US GDP data and must display this data in a [time series bar chart](http://codepen.io/jstoebel/pen/GjJaGA).

There are plenty of tutorials out there on D3 and even more examples where it is easy enough to copy/paste/change variables and move on. If I was casual user of D3, looking for a quick solution, I might go with the copy/paste solution. However, I want data visualization to be a core competency in my tool belt, so I want to understand D3 deeply. There was one topic in particular that I wrestled with: scales. Here's a quick run down of what they do.

## Scales

I wrestled with this concept a lot on this project. Essentially, the scale maps a series of input values to how they should be displayed visually. Here is a simple example:

```
var yScale = d3.scale.linear()
        .domain([0, 100])
        .range([0, 500]);
```

In this example, we are saying we are inputing data from 0 to 100 and expect an output of 0 to 500. An input of 1 expects an output of 5, an input of 2 expects 10 etc. To express this value mathematically, `y = 5x`. Scales were a very confusing, abstract concept for me at first, but think of it as an expression of the relationship between input data how those values should be represented. This is a common concept for developers: its what all good functions do! In the code snippet above we are merely building up a function that we can later use when drawing our chart, like so:

```
var myChart = d3.select("#chart").append('svg')
      .style('background', '#E7E0CB')

      ...

        .selectAll('rect').data(inputData)
        .enter().append('rect')
          .attr('height', function(item) {
            return yScale(item);
          })
```

Here, as I a drawing the chart, I am using the yScale function that I built up earlier to determine the height of each bar as a function of its corresponding data input. So if the variable `inputData` was `[1,2,3,4]`. The height of the bars would be, respectively, `[5, 10, 15, 20]`

## Time Scales
But what if our scale measures a series time or date data points? Reasoning about dates and times can get hairy fast, but fortunately D3 has a special scale to take care of it for you. In this example I have an array of financial quarters from 1947 to 2015:

```
var quarters = ["1947-01-01, "1947-04-01", ... "2015-12-01"];
var xScale = d3.time.scale()
        .domain([d3.min(quarters), d3.max(quarters)])
        .range([0, width])
```

Why is this so cool? Because D3 automatically understands each of the items in `quarters` as dates and can reason about them accordingly. In [this example](http://codepen.io/jstoebel/pen/GjJaGA), D3 looks at the range of dates and determines that the axis would be best represented by decade. If, in a different example we had data points for every day in a single month, the axis would automatically adjust, with ticks marking days. Neat, huh?
