---
layout: post
title: "Customize kali linux"
date: 2014-05-24 17:39:39 +0200
comments: false
categories: 
---
### A quick guide to setup kali linux with some extras. 
As I just got myself a ultrabook and wanted a linux
distribution to experiment with security setups and some development. 
The installation is a normal [kali linux](http://www.kali.org/downloads/)
 with customized tmux, ruby on rails develop envoronment.

Item that need to be instaled are:

 - sublime (text editor)
 - Redis (fast key, value database)
 - nodejs (javascript runtime)

#### tmux
change default b to a for easier keybinding, the tmux.conf can be found here
[tmux.conf](https://github.com/chrjoh/dotfiles/blob/master/kali_tmux.conf)


#### git
So we can have version controll, follow the instructions on github
[ssh key setup](https://help.github.com/articles/generating-ssh-keys)

#### Sublime
Download sublime2 [here](http://www.sublimetext.com/2) or sublime2 [here](http://www.sublimetext.com/3)
I choose sublime2, then extract the package with :
`tar xf Sublime\ Text\ 2.0.1\ x64.tar.bz2`

You’ll get a “Sublime Text 2″ folder after extraction. This folder contains all the files that Sublime Text will need. Then I moved it to “/opt/” folder :
`mv Sublime\ Text\ 2 /opt/`

To be able to use it from a terminal with “sublime”. For that just create a symbolic link in “/usr/bin” :
`ln -s /opt/Sublime\ Text\ 2/sublime_text /usr/bin/sublime`


#### Redis
Just download the 64-bit verion 
[here](http://download.redis.io/redis-stable.tar.gz) with 
`wget http://download.redis.io/redis-stable.tar.gz` there is also a online guide 
[here](http://redis.io/topics/quickstart) to unpack use 
`tar xvzf redis-stable.tar.gz` change directory and comåile with `make`
Do not forget to run `make test` after, to install run `make install` this will just copy the binary files
so that you can start Redis then needed. To install it as service check the file README


#### NodeJS
This is a Javascript runtime that is needed to run various rails applications, I added the debian repository
in `/etc/apt/sources.list` just add this line :
`add deb http://ftp.de.debian.org/debian sid main`
and run the following commands `apt-get update` followed by `apt-get install nodejs`