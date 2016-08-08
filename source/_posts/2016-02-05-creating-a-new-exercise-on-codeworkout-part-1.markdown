---
layout: post
title: Creating a new exercise on CodeWorkout-Part 1
date: '2016-02-05T13:38:02+00:00'
permalink: creating-a-new-exercise-on-codeworkout-part-1
---
###TL;DR
<p>I haven&#8217;t quite figured out how to create a new exercise, but I&#8217;ve learned more about the architecture and am very close.</p>
<p>Our first contribution to CodeWorkout will be to create a new workout which is a collection of programming exercises. The app doesn&#8217;t yet have a UI for professors or admins to add these directly in their browser (this could be a great project for later). For the mean time, my goal is to workout how to do this from the command line. While this is probably the least practical way to add content to an app, this exercise sheds a little light onto how the whole thing is put together.</p>
<p>After getting CodeWorkout <a href="https://jacobstoebel.wordpress.com/2016/01/31/getting-codeworkout-set-up-on-localhost/" target="_blank">running on a local machine</a> let&#8217;s browse to this URL: <a href="http://localhost:3000/gym/exercises/1/practice" target="_blank">http://localhost:3000/gym/exercises/1/practice</a> to take a peek at a question. We see an exercise in Java as follows &#8220;Write a function in Java called <code>factorial()</code> that will take a positive integer as input and returns its factorial as output.&#8221;</p>
<p>So where does this exercise come from? To figure this out, let&#8217;s trace this page from its initial request, through the routes file, into the controller, the models that are queried and finally the view that gets rendered.</p>
###Router
<p>If I type <code> rake routes </code>into my terminal from the root directory of this application I can see all of the url paths that this application knows how to fulfill (for a more user friendly version of this I can also visit <a href="http://localhost:3000/rails/info/routes" rel="nofollow">http://localhost:3000/rails/info/routes</a>.) These routes are examined from top to bottom against the requested URL. When a match is found, the corresponding controller and action is called. About halfway down the page I find an entry that seems to match my url:</p>

    exercise_practice GET     /gym/exercise/:id/practice(.:format)  exercises#practice
 
<p>This means that if I issue a GET request against this URL pattern (and I did) I will be routed to the exercises controller and the practice action. And by the way, that symbol :id means that I can stick any number in there. This corresponds to the exercise id in the database. <a href="https://www.youtube.com/watch?v=Yic7IRO9d6I" target="_blank">To the exercises controller!!</a></p>
###Exercise Controller
<p>The controller is the part of the app responsible for responding to requests made against a particular resource (here an exercise), setting up the instance variables needed and then rendering the appropriate view. Exercise#practice is large action but thankfully not all of it will apply to this particular request.  If I look on line 409 I see that an Exercise object is being created by looking up a record based on the :id that was passed into the URL. A little later on line 422, we assign the <code> @exercise_version</code> by calling the method <code>.current_version</code> on the exercise. It&#8217;s hopefully intuitive what this method does: the database is structured to allow for multiple versions of a particular exercise. Calling <code> current_version </code> gives us the most recent one (i.e. the one that should be shown to the student). So where is that method? We don&#8217;t see it anywhere in this file. The answer is in the model.</p>
###The Model
<p>Looking at the model file exercise.rb we see line 59 <code> belongs_to :current_version, class_name: 'ExerciseVersion' </code> This is us telling Rails (and ActiveRecord, the gem which interacts with our database) that there is a one to many relationship between the exercise table and the exercise_versions table, and that we are setting a method called current_version on the Exercise class to find the latest version related to this exercise.</p>
###The View
<p>Ok, so the controller, among many other things, sets up the requested exercise and fetches the most current version and then renders the related view located at app/views/exercises/practice. The resulting .haml file is a bit hard to read if you haven&#8217;t seen it before, but its basically html with ruby embedded in it. Rails knows how to run the Ruby and then send to the user a simple html file that gets rendered in the browser. Again, there is a lot going on here, but looking at it, we can see some of the same instance variables that were set in up in the controller.</p>
<h2>Conclusion</h2>
<p>While all of this code looks confusing, this quick dive in gives me a bit of hope that we can figure it out. For my next post coming soon, I will write up how to create a new record. We&#8217;re almost there.</p>
