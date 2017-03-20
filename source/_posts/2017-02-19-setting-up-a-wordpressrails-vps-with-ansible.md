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

There were however a few small issues that caused me a disproportionate level of confusion.

 1. It turns out that 256 Mb of memory isn't enough to install MySQL. You will likely get the error: `invoke-rc.d: initscript mysql, action "start" failed.
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

 2. You'll also notice in above that I am using `bento/ubuntu-16.04` as the base kernel. The screencast uses `ubuntu/trusty64` but I wanted to use xenial. The problem here is that this kernel does not set up the assumed default user/password as `vagrant` which throws a wrench into some things. Rather than customize it, I just switched to bento and everything works fine.

 - trouble installing mysql with `geerlingguy.mysql` getting this error

```
invoke-rc.d: initscript mysql, action "start" failed.
dpkg: error processing package mysql-server-5.5 (--configure):
 subprocess installed post-installation script returned error exit status 1
dpkg: dependency problems prevent configuration of mysql-server:
 mysql-server depends on mysql-server-5.5; however:
  Package mysql-server-5.5 is not configured yet.

dpkg: error processing package mysql-server (--configure):
 dependency problems - leaving unconfigured
No apport report written because the error message indicates its a followup error from a previous failure.
                                      Errors were encountered while processing:
 mysql-server-5.5
 mysql-server


 Unable to set password for the MySQL "root" user              │
 │                                                               │
 │ An error occurred while setting the password for the MySQL    │
 │ administrative user. This may have happened because the       │
 │ account already has a password, or because of a               │
 │ communication problem with the MySQL server.                  │
 │                                                               │
 │ You should check the account's password after the package     │
 │ installation.                                                 │
 │                                                               │
 │ Please read the                                               │
 │ /usr/share/doc/mysql-server-5.5/README.Debian file for more   │
 │ information.  
```
