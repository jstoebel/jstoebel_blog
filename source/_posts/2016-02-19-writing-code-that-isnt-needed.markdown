---
layout: post
title: Being productivly lazy, or not writing code that isn't needed
date: '2016-02-19T00:28:08+00:00'
permalink: writing-code-that-isnt-needed
---
Here is a neat video from Google on [writing code that isn't needed][1]

I was reminded of several experiences writing new code where I very quickly lost track of the problem I needed to solve. Very often this includes assuming corner cases that simply don't exist, or building features that assume the problem is more complex than it in fact is.

I frequently practice my programming skills on [Codewars.com][2]. A while back I took on a relatively simple exercise in Python (or kata as they are called on that site) that parsed a time delta or a measure of distance between two times (example 1:02:15) and returned it in a human readable format (1 hour, 2 minutes, 15 seconds).

I immediately dove into the challenge setting out to make it more complex than it really needed to be. For example, I implemented a fully functional `pluralize` function which took a quantity, a word in its singular form and how the plural form is spelled should it not be a simple matter of tacking an 's' on the end. What I failed to consider was that it _was_ a simple matter of tacking an 's' on the end since I was only ever dealing with the words 'years', 'days', 'hours', 'minutes', 'seconds'. If octopi and cacti were units of time, it would be a different matter. But alas they aren't. They really really aren't. 

Why do we do bone-headed stuff like this? Is it because we get into code monkey mode and just start banging out code that solves _a_ problem but not the problem at hand? Yes. Is a part of our brains still in high school where we think we can get extra credit for addressing problems that don't actually exist? Absolutely. We programmers are smart people, but that doesn't mean every challenge needs to stretch our brain cells to their capacity. As the video points out at one point, there is often value in thinking less.

PS: I think the Zen of Python by Tim Peters speaks to this topic more than any other piece of writing I know of:

> 
    The Zen of Python, by Tim Peters
>
    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than right now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!

You can find this by typing `import this` into your Python interpreter. It actually breaks many of its own rules in its implementation (programmer humour?)

    s = """Gur Mra bs Clguba, ol Gvz Crgref

    Ornhgvshy vf orggre guna htyl.
    Rkcyvpvg vf orggre guna vzcyvpvg.
    Fvzcyr vf orggre guna pbzcyrk.
    Pbzcyrk vf orggre guna pbzcyvpngrq.
    Syng vf orggre guna arfgrq.
    Fcnefr vf orggre guna qrafr.
    Ernqnovyvgl pbhagf.
    Fcrpvny pnfrf nera'g fcrpvny rabhtu gb oernx gur ehyrf.
    Nygubhtu cenpgvpnyvgl orngf chevgl.
    Reebef fubhyq arire cnff fvyragyl.
    Hayrff rkcyvpvgyl fvyraprq.
    Va gur snpr bs nzovthvgl, ershfr gur grzcgngvba gb thrff.
    Gurer fubhyq or bar-- naq cersrenoyl bayl bar --boivbhf jnl gb qb vg.
    Nygubhtu gung jnl znl abg or boivbhf ng svefg hayrff lbh'er Qhgpu.
    Abj vf orggre guna arire.
    Nygubhtu arire vf bsgra orggre guna *evtug* abj.
    Vs gur vzcyrzragngvba vf uneq gb rkcynva, vg'f n onq vqrn.
    Vs gur vzcyrzragngvba vf rnfl gb rkcynva, vg znl or n tbbq vqrn.
    Anzrfcnprf ner bar ubaxvat terng vqrn -- yrg'f qb zber bs gubfr!"""

    d = {}
    for c in (65, 97):
        for i in range(26):
            d[chr(i+c)] = chr((i+13) % 26 + c)

    print "".join([d.get(c, c) for c in s])


  [1]: https://youtu.be/JOAq3YN45YE
  [2]: http://www.codewars.com
