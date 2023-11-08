---
author: kevinkoo001@gmail.com
comments: false
date: 2014-07-05 16:26:05+00:00
layout: post
link: http://dandylife.net/blog/archives/244
slug: how-to-transfer-putty-setting-to-linux
title: How to transfer PuTTY setting to linux
wordpress_id: 244
categories:
- Attack &amp; Defense, Cyber Warfare
tags:
- Tips
---

Putty is a simple but strong (and even free) terminal to connect to remote machines. But the settings in windows are stored in the registry (i.e. HKU\Software\SimonTatham\PuTTY\Sessions). There was a guy who made the script to transfer the configuration to linux! Here are the steps to follow:

1) Backup your PuTTY sessions at the *.reg type to somewhere.

    
    C:\ regedit /e "%userprofile%\desktop\putty-sessions.reg"


2) Download and install your PuTTY in your box: [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html ](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)

    
    # wget http://the.earth.li/~sgtatham/putty/latest/putty-0.63.tar.gz
    # tar zxvf putty-0.63.tar.gz 
    # cd putty-0.63/
    # ./configure
    # make; make install


3) Download the script (i.e pwin2lin.pl) from here: [https://pwin2lin.googlecode.com](https://pwin2lin.googlecode.com)

    
    # wget https://pwin2lin.googlecode.com/files/pwin2lin.pl


4) Make sure you have the following directory: ~/.putty/sessions

5) Execute the script and enjoy!

    
    # perl pwin2lin.pl ./putty-sessions.reg ~/.putty
    # ls -l ~/.putty/sessions/
    total 72
    -rw-r--r-- 1 root root 4237 Jul  5 11:32 G*****Started
    -rw-r--r-- 1 root root 4230 Jul  5 11:32 K***
    -rw-r--r-- 1 root root 4210 Jul  5 11:32 **-NAS
    -rw-r--r-- 1 root root 4269 Jul  5 11:32 ***Server
    -rw-r--r-- 1 root root 4220 Jul  5 11:32 n***a
    -rw-r--r-- 1 root root 4224 Jul  5 11:32 ***ws1
    -rw-r--r-- 1 root root 4265 Jul  5 11:32 ***ws1-linux
    -rw-r--r-- 1 root root 4210 Jul  5 11:32 R***
    -rw-r--r-- 1 root root 4225 Jul  5 11:32 ***S
    # putty&


(Reference) [http://mdinh.wordpress.com/2013/06/07/migrating-windows-putty-registry-to-linux/](http://mdinh.wordpress.com/2013/06/07/migrating-windows-putty-registry-to-linux/)
