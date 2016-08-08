---
layout: post
title: 'Q&A forum: breaking things down'
date: '2016-03-01T00:44:44+00:00'
permalink: qa-forum-breaking-things-down
---
Today in class we gave our [lightning talk][1] on version 0 of the Q&A forum. We also drafted our [SDS][2] on the topic. In class on Wednesday, we will begin to write our battle plans for moving forward on version 0 and beyond.  Here is my thinking on how we might break up the work into manageable chunks.

##First, the infrastructure
The first thing I think we should do (maybe Wednesday in class) is to generate the skeleton of all of the files, classes and methods we'll need to lay the ground work for version 0. We'll need:

* A controller for both our resources (`questions` and `responses`). We can use `rails generate` for this
* Models tables for both of our resources. `rails generate model` will make our models along with a migration file which we can use to create the database tables. We should also associate our models with one another using `has_many` and `belongs_to` 
* We'll need to edit the routes file to allow for the routes specified in our SDS. [Restful routes please!][3] 
* Finally, we'll need to add html.haml files for each of the views we expect.

##Agree on pre/post conditions
Next, we might want to agree on pre and post conditions for each controller action. We don't need to write the code all together, but we might want to write comments, outlining what params come into each action, what variables are created and what views are rendered.

##Divide and conquer
After setting up all of the above, we can work in pairs or individually to take responsibility for each page. People responsible for a page would also take responsibility for writing the corresponding controller action as well as any additional private methods, JavaScript, etc.

##What about styles?
It might be useful if someone wants to propose to the group some unifying styles for the Q&A forum. This will hopefully draw largely on the aesthetic of the CodeWorkout site at large as well as some of our own.

##Frequent code reviews
I think it could be helpful if we all push our work to separate branches, review it when we can all get together (maybe during our weekly meetings) and merge it in when we feel ready. Since we are a relatively small group working closely together, I think it would be super helpful if we tried to look at each other's work closely.

  [1]: https://docs.google.com/presentation/d/1GI3NAWHgcnjVdGQdE9kQNCARnFI6L1EtMDfvUvDfHrE/pub?start=false&loop=false&delayms=3000
  [2]: https://docs.google.com/document/d/1zRjPPrgYsVRvo2fCCvt24O0Nt-XAqJbHs7h_giXlWGs/edit?usp=sharing
  [3]: http://guides.rubyonrails.org/routing.html
