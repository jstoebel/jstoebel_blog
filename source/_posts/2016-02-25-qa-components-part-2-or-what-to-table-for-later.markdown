---
layout: post
title: Q&A part 3, or what to table for later
date: '2016-02-25T21:54:42+00:00'
permalink: qa-components-part-2-or-what-to-table-for-later
---
In my last [blog post][1] I explored the components I believe we need for a version 0 of the Q&A forum. Today I am thinking about what might come after that. There are a number of features that _could_ be exciting to include, but in the interest of getting a prototype up and working, we should table them for now. None the less, it is important to keep track of these ideas for future implementation.

 - The ability for professors to up vote certain answers. This could be a helpful feature for students as it gives the student asking the question a certain level of confidence in the answer. To do this, we would need an additional attribute in the Response Model (`instr_approved` perhaps) and a new action in the responses controller. The user could click a star or check mark to fire an AJAX request to approve the response

- Participation points?: Maybe students can earn exp points for answering questions (and even more points for getting a response stared). Students who earn enough points through completing exercises and/or participating in the Q&A are granted the ability to endorse responses themselves (though not their own of course). Here we would have various controller actions altering a student's experience points attribute.

- Badges: In addition to a point system, Stackoverflow also has a number of badges awarded to users for participating in positive behaviours. Why not do the same? This would involve a two new tables: `badges` which is a listing of all badges available and `student_badges` which maps badges earned by each student. 


  [1]: http://www.jstoebel.com/blog/problem-decomposition-building-a-qa-forum/
