---
layout: post
title: "Diving into Typescript"
permalink: diving-into-typescript
date: 2018-09-17 15:48:44
comments: true
description: "Diving into Typescript"
keywords: ""
categories:

tags:
  - Typescript
  - Apollo

---

I have a confession: I think I like Typescript.

As someone coming from the world of dynamic languages, I've sort of just assumed that I preferred getting to avoid the ceremony of declaring types as they are created, passed into and returned from functions. I say assumed because apart from a short experience learning c++ (didn't go well) I don't really have much experience with strictly typed languages. For all of my career as a developer so far, I've been fine with this lack of knowledge. From the early does as a beginner you get used to the kinds of errors you can expected. When you see something like `undefined is not a function` in javascript you say to yourself "Oh! I'm try to call a function that isn't attached to that object." and you gain instincts about where the problem may have arising (hopefully). You also write a tests to give you confidence that the data moving through your system is of the correct shape. Being a Rubiest as well, I learned the benefit of not caring about types: "we don't _actually_ care about the type of object we're working with. _Really_ what we care about is what messages it responds to behavior it exhibits." I still love my dynamic languages as much as ever, but I am beginning to appreciate the value in strict typing as well.

For example, the appeal to using React is the ability to create small, reusable components. As your project begins to scale, one common source of bugs can happen when components don't enforce a strict contract for how they should be created, the shape of data coming in from the backend, or how data should be passed internally between methods. Typescript (along with the alternative Flow) offer some more structure that I've been craving. 

But there has been a bit of a learning curve as well. For example, I've been using Apollo client my new electron project. At one point I wanted to pass a query to the cache:

```
const {allTasks} = cache.readQuery({ query: ALL_TASKS });
```

This is pretty much how it works based on the [official tutorial](https://www.apollographql.com/docs/react/essentials/mutations.html). Not so fast with Typescript though! We get the error `Type '{ allTasks: any; } | null' has no property 'allTasks' and no string index signature.`

Trying it a slightly different way we get 

```
const allTasks = cache.readQuery({ query: ALL_TASKS }).allTasks; // Object is possibly 'null'.
```

What's going on here? I think about it for a bit an get the basic idea: we are issuing a query that, at compile time we have no idea of knowing what kind of data will be returned. I [asked a question](https://stackoverflow.com/questions/52348098/typescript-apollo-access-property-from-in-memory-cache/52350196?noredirect=1#comment91686844_52350196) on Stackoverflow, and with a little help came to understand that `readQuery` accepts a type argument describing the shape of the data you expect to be returned. What about the compiler complaining about `null` though? This part may be either a bug or a mistake in documentation as [according to this](https://www.apollographql.com/docs/react/advanced/caching.html#readquery) `readyQuery` should throw an error if no results are found.