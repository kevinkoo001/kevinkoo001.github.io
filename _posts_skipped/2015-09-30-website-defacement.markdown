---
author: kevinkoo001@gmail.com
comments: false
date: 2015-09-30 00:19:50+00:00
layout: post
link: http://dandylife.net/blog/archives/531
slug: website-defacement
title: Website defacement
wordpress_id: 531
categories:
- Attack &amp; Defense, Cyber Warfare
- Miscellaneous Stuff
---

I bumped into a 'hacked' page while checking my page this morning.  I found the following script has been inserted into all WordPress-based pages. Also, the title was altered by a hacker. I was able to find similar attacks around the globe.

    
    <script>document.documentElement.innerHTML = unescape('%48%61%63%6b%65%64%20%42%79%20%78%2d%53%68%6f%6e%61');</script>


Because I have updated into the latest version of WordPress since last week, I doubt it was caused by WordPress vulnerability itself.  After I looked into web logs, other traces, and related attacks, it looks like the hosting server has been compromised recently. The attack looks quite similar to '_Hacked by Badi_' a few years ago.

For those who have gone through the defacement, the following link would help. (Hopefully the server should be patched immediately.)

[https://wordpress.org/support/topic/hacked-by-badi-1](https://wordpress.org/support/topic/hacked-by-badi-1)
[http://whmscripts.net/misc/2013/apache-symlink-security-issue-fixpatch/](http://whmscripts.net/misc/2013/apache-symlink-security-issue-fixpatch/)
