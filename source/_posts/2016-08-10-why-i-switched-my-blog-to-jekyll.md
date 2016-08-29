---
layout: post
title: "Why I switched my blog to Jekyll"
permalink: why-i-switched-my-blog-to-jekyll
date: 2016-08-10 08:32:02
comments: true
description: "Why I switched my blog to Jekyll"
keywords: ""
categories:

tags:

---

Earlier this year I launched my personal site running on Django and the Mezzanine CMS. Things were going relatively well. The hosting provider [Python Anywhere](https://www.pythonanywhere.com) made the set up painless. But just last week I moved everything to a Github pages site with Jekyll. Here's why:

# Lightweight

Jekyll doesn't have you worrying about nonsense like web server configuration or databases. And why should it? I'm just trying to push static content out to the web. I simply write my post as a markdown file (<3), push to Github and I'm done. Anything more complex amounts to more needless headache. With my old site I also had to worry about backup and security. With Github pages, backup is a simple matter of backing up my hard drive (which I do anyway). As far a security, there are no application account passwords to fuss over or SSL certificate to set up. I publish my work to my Github repo using my RSA key like normal and that's it. Visitors view my site with plain old http, but since I'm not asking them to submit any data, is that really a problem?

# Common sense conventions...

Jekyll ships with several common sense conventions straight out of the box. When Github goes to build your page it looks for source material and config in common sense locations. Basically, it just works the way you would expect it to. When you do have questions there are plenty of easy to follow solutions out there.

# ...with plenty of room for customization

But there's also plenty of room to play and do some amazing things. Currently, my site is a fork from [here](https://github.com/nandomoreirame/nandomoreira-jekyll-theme) who has incorporated lots of front end magic. Jekyll effectively wins both sides of the argument between simplicity and flexibility. Its easy to set up, but also gives you the confidence that there is plenty you can tweak if you want to later.
