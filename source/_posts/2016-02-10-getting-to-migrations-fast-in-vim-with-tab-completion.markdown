---
layout: post
title: Getting to migrations fast in vim with tab completion.
date: '2016-02-10T15:16:24+00:00'
permalink: getting-to-migrations-fast-in-vim-with-tab-completion
---
<p>Migrations in rails are pretty neat, but one frustration is that pulling them up from the command line is a pain. Migration files names are perpended with a time stamp like so <code> 20160209220544_first.</code> Rails needs to do this in order to know how to traverse them in order. But their not super useful if your trying to tab complete. There&#8217;s a better way!</p>
<p>Inside vim you can type <code> :find path/to/rails/root/db/migrate/ </code> and then press tab to cycle through the possible choices. Sweet! But it gets even better. What if you have a long path to your root directory or can&#8217;t remember it? You can ask vim to always start off from the directory you entered in unix. To do that make a new file in your home directory called .vimrc and add the line <code> set autochdir </code> #takingcareofbussiness</p><br />  <a rel="nofollow" href="http://feeds.wordpress.com/1.0/gocomments/jacobstoebel.wordpress.com/842/"><img alt="" border="0" src="http://feeds.wordpress.com/1.0/comments/jacobstoebel.wordpress.com/842/" /></a> <img alt="" border="0" src="https://pixel.wp.com/b.gif?host=jacobstoebel.wordpress.com&#038;blog=104465458&#038;post=842&#038;subd=jacobstoebel&#038;ref=&#038;feed=1" width="1" height="1" />
