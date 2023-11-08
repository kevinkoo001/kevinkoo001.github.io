---
author: kevinkoo001@gmail.com
comments: true
date: 2014-02-17 04:03:44+00:00
layout: post
link: http://dandylife.net/blog/archives/181
slug: changing-the-directory-color-in-the-bash-shell
title: Changing the Directory Color in the Bash Shell
wordpress_id: 181
categories:
- Miscellaneous Stuff
tags:
- Bash
- Directory color
- Linux
---

While running Linux distribution,  I often found it that I could hardly see directory names due to dark color (well, it is dark blue in black background, which is hard to believe why this combination is chosen)

Thanks to [this posting](http://www.geekgumbo.com/2011/11/04/changing-the-directory-color-in-the-bash-shell/), I could change the color. Here's a simple way.

1. Open .bashrc in your home directory.
2. Create the following entry. Check out the [value] below. The first number is the degree of darkness, followed by a semicolon, and then the actual number of the color.

    
    LS_COLORS='di=[value]' ; export LS_COLORS
    
    <strong>[value]</strong>
    Blue = 34
    Green = 32
    Light Green = 1;32
    Cyan = 36
    Red = 31
    Purple = 35
    Brown = 33
    Yellow = 1;33
    white = 1;37
    Light Grey = 0;37
    Black = 30
    Dark Grey= 1;30


3. Restart bash shell.
