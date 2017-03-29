---
layout: post
title: "setting up a wordpress/rails vps with ansible"
permalink: setting-up-a-wordpressrails-vps-with-ansible
date: 2017-02-19 15:56:58
comments: true
description: "setting up a wordpress/rails vps with ansible"
keywords: ""
categories:

tags:

---

For my latest project with [Mountain Tech Media](http://www.mttechmedia.com/) I decided to take some extra time use Ansible to set up my VPS. I looked at Ansible as well as Chef, and while I don't pretend to be an expert on either, I found Ansible to be a bit more friendly for beginners with small projects who just want to get up and running. I found [this tutorial](https://sysadmincasts.com/episodes/45-learning-ansible-with-vagrant-part-2-4) to be incredibly helpful in getting a Vagrant development environment setup. Check it out.

There were however a few small issues that caused me a disproportionate level of confusion in getting things setup:

### how much memory?

It turns out that 256 Mb of memory isn't enough to install MySQL. You will likely get the error: `invoke-rc.d: initscript mysql, action "start" failed.
 dpkg: error processing package mysql-server-5.5 (--configure):`. To fix this, I increased memory allocation to 1Gb:

{%highlight ruby %}

config.vm.define :node do |node|
    # the droplet to provision
    node.vm.box = "bento/ubuntu-16.04"
    node.vm.hostname = "web1"
    node.vm.network :private_network, ip: "10.0.15.11"
    node.vm.network "forwarded_port", guest: 80, host: 8080
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
end

{% endhighlight %}

### Which Kernel

You'll also notice in the above that I am using `bento/ubuntu-16.04` as the base kernel. The screencast uses `ubuntu/trusty64` but I wanted to use xenial. The problem here is that the `ubuntu/xenial` kernel does not set up the assumed default user/password as `vagrant` which throws a wrench into some things. Rather than customize it, I just switched to bento and everything works fine.

### Configuring Nginx
__note: part of the following section is a copy/paste of my own words originating from [here](stackoverflow.com/questions/42521316/rails-5-nginx-passenger-cant-serve-static-assets-404?noredirect=1#comment72180338_42521316)__

I am using passenger to serve my app, but letting nginx serve my precompiled assets. I spent a lot of time confused by how to make this work. Here is the config file I am using in `/etc/nginx/sites-enabled/default`

{% highlight %}

    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root {{wp['root_dir']}};
        index index.php index.html index.htm index.nginx-debian.html;

        passenger_enabled on;
        passenger_app_env production;
        server_name {{ ansible_eth0["ipv4"]["address"] }};

        location / {
            #try_files $uri $uri/ =404;
            try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }

        location ~ /\.ht {
            deny all;
        }

        location = /favicon.ico { log_not_found off; access_log off; }
        location = /robots.txt { log_not_found off; access_log off; allow all; }
        location = .(css|gif|ico|jpeg|jpg|js|png)$" # don't include this line! see below!

        location ~ ^/h20-initiative(/.*|$) {
            alias /home/{{user_name}}/unify/current/public$1;
            passenger_base_uri /h20-initiative;
            passenger_app_root /home/{{user_name}}/current/unify;
            passenger_document_root /home/{{user_name}}/unify/current/public;
            passenger_enabled on;

        }

        location ~ ^/assets/ {
            expires 1y;
            add_header Cache-Control public;

            add_header ETag "";
            break;
        }

    }

{% endhighlight %}

Here I am pointing requests for `/` to my Wordpress install (the variable wp['root_dir'] determines this.) and requests to `/h20-initiative/*` to to my rails app.

I also ran into a strange problem where static assets in my rails app were not being served. After a good deal of head scratching the Googling around, I was given some advice on how to grep through the nginx log. First I set my nginx log level to `debug`. To do that, go to `/etc/nginx/nginx.conf`. Find the line `error_log /var/log/nginx/error.log;` and change it to `error_log /var/log/nginx/error.log debug;`

Then I greped through the log. I searched for one of the assets that wasn't being served and found these lines:

{% highlight %}
2017/03/01 02:02:31 [debug] 28274#28274: *231 test location: "/"
2017/03/01 02:02:31 [debug] 28274#28274: *231 test location: "robots.txt"
2017/03/01 02:02:31 [debug] 28274#28274: *231 test location: "favicon.ico"
2017/03/01 02:02:31 [debug] 28274#28274: *231 test location: ~ "\.php$"
2017/03/01 02:02:31 [debug] 28274#28274: *231 test location: ~ "/\.ht"
2017/03/01 02:02:31 [debug] 28274#28274: *231 test location: ~ "\.(css|gif|ico|jpeg|jpg|js|png)$"
2017/03/01 02:02:31 [debug] 28274#28274: *231 using configuration "\.(css|gif|ico|jpeg|jpg|js|png)$"

{% endhighlight %}

It looks like the block `"\.(css|gif|ico|jpeg|jpg|js|png)$"` was the matched block and used to determine where to find requests for static assets. The problem is, that's not where those files are! In fact, I don't need to serve any assets from the root directory. I deleted that block, allowing for the intended block to serve my assets.

### User installed plugins themes and updates.

I want my admin users to have permission to install plugins, themes and updates. We only have two admin users and I work closely with them to ensure nothing malicious is installed. We struggled a bit with this ability until I realized that permissions in the WP install were not done correctly. The owner of the process running Nginx needs permission to read and write to the directories. I did this by granting 0770 permission to all files and directories in the wordpress install.

### Running rails app at sub path

The main site will be served from `/` and the rails app from /h20-initiative. This presents a somewhat tricky problem in how to get urls to point to the right place. For example I have some Javascript that points to an end point in my app:

{% highlight javascript %}

d3.json("/places/active", function(error, activePlaces){
  // do something with the result
}
{% endhighlight %}

This works perfectly fine in development but in production the endpoint is at `/h20-initiative/places/active`. How do we consolidate this? I _could_ do all of my app development inside of a vagrant box that's running both my Wordpress install and this app. But that's an awful lot of overhead just to solve a name spacing issue when all I'm trying to do is hack on a rails project and ignore the rest. Instead, I just stored the apps' relative path in an environment variable and then prepend it in paths that need it. For example:

In my Javascript...

{% highlight javascript %}

// in the view attach a variable to the `window` object:
:javascript
    window.base_url = "#{ENV["BASE_URL"]}";

// then access it in the javascript

d3.json(base_url + "/places/active", function(error, activePlaces){
  // do something with the result
}
{% endhighlight %}

For all of my route helpers I can add the relative path in `config/application`

{% highlight ruby %}

Bundler.require(*Rails.groups)
module Unify
  class Application < Rails::Application
    # ...
    config.relative_url_root = ENV["BASE_URL"]

  end
end

{% endhighlight %}

In scss I can use an erb tag and `image_url` to reference an image I want for a
background

{% highlight %}

$bg-img: "<%= image_url 'cumberlands.jpg' %>";

body{
  background:url($bg-img);
  // ...
}

{% endhighlight %}

And that's it! So much easier than having to change my development enviornment!
