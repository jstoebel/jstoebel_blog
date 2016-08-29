---
layout: post
title: Learning Async in JavaScript
date: '2016-06-01T12:30:17+00:00'
permalink: learning-async-in-javascript
---
Recently, I completed Freecodecamp's [Front End Development Program](https://www.freecodecamp.com/jstoebel). The curriculum is in Javascript meaning that callbacks and promises were on the menu for me to wrestle with. There were plenty of great resources out there to help new developers with these challenges, but as part of my learning, I thought it would be useful to summarize some of the lessons I learned. At the very least, this will help solidify my learning. And it might even be a useful quick resource to others as they walk the same path I do.

## Async

Coming from Python, asynchronous operations was a tricky topic to get my head around. Here's the basic idea: Let's say you execute a command that doesn't finish instantly. You might be setting a timer, or executing an AJAX request. Typically, a callback is given to that command; something that should be done once that command is finished. So for example you might have:

    Do a thing
    Do another thing
    Set a timer for 5 seconds
        when its down do a thing
    Do a final thing.


A beginner coming from Python would be forgiven for thinking that the line `do a final thing` would run last right? Not quite. The confusing, but ultimately great thing is that JavaScript doesn't just wait around for a a timer to end. There's other work to be done! Once the timer has been set `Do a final thing` runs right away, then there is a pause and then the timer's callback is called. Think about it this way: if you were cleaning your house, when you put some laundry in the washing machine, you wouldn't just sit there while the cycle runs waiting for it to finish. You would probably go work on something else for a while. When the cycle finishes the "callback" runs, which means you take the clothes out of the machine and hang them up to dry. As a more real world example, I was working on a clone of the game [Simon](https://www.freecodecamp.com/jstoebel). The game needs to playback a series of flashes to the user, but there needs to be a gap of time in between the flashes to allow for the user's brain to process the information. This means that we need to set two timers. One to trigger the end of a flash and another to trigger the beginning of the next (with a gap of time in the middle). We might be tempted to write psudocode like this:

    loop over each move:
        turn on button
        set a timer for 500 ms:
            when its done, turn off the button
            set a timer for 200 ms:
                when its done flash the next button

I knew I had an issue with async when this code didn't work as I expected. Again, once Javascript sets a timer it does not wait for it to finish. It keeps going with the next line of code, which in this case means the next iteration of the for loop. This isn't what we want.

###Use a generator!

This seemed like a perfect place to use a generator. A generator is an object that knows how to produce a series of values. Here is what I came up with:

    this.playback = function(){
        //playback the current set of moves.
        var game = this;
        function* getMove(){
            var mi = 0;
            while(mi < game.moves.length){
                yield game.moves[mi]
                mi++
            }
        }

        var moveGen = getMove();
        this.playbackFlash(moveGen);
    }

    this.playbackFlash = function(moveGen){
        //flashes button, plays sound and sets timer. When timer goes off, decide if it should call again.
        //moveGen: generator that traverses the moves array
        //this function is recursive. It uses a generator to determine the next item in the array to act on.
        //when we fall off the end of the array, the method will stop recursing.

        $("simon-btn").removeClass("flash");
        $(".simon-btn").off("click")
        var move_obj = moveGen.next()
        if(move_obj.done == false){

            var move = move_obj.value;
            console.log("flashing "+move);
            var btn = "#btn"+move;
            this.flash(move)
            this.playCorrectSound(move);
            var game = this;
            setTimeout(function(){
                //timer to end current flash
                game.endFlash(move)
                setTimeout(function(){
                    //timer before next flash
                    game.playbackFlash(moveGen)
                }, 200)
            }, this.speed)
        } else {
            this.takeGuess();
        }
    }

These are the two methods for playing back the sequence of flashes to the player. The method `playback` creates a generator that knows how to traverse `moves` which is just an array of integers corresponding to the button to flash. The generator is passed to `playbackFlash`. For each call to `playbackFlash` we call `.next()` on the generator which produces the next series in the array. We can then turn the button on, set a timer to turn it off, and then set a timer for 200 ms before the next flash. When that second times goes off, our callback can call `playbackFlash` again passing in the same generator. Because the generator preserves its state, when it has reached the end of its collection the function's if statement will fail and the function will pass control to another part of the program (the part where we handle the player's input.)
