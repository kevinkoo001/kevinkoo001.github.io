---
author: kevinkoo001@gmail.com
comments: false
date: 2016-01-21 22:22:19+00:00
layout: post
link: http://dandylife.net/blog/archives/565
slug: reverse-engineering-resources
title: Assembly visualizer for reversing
wordpress_id: 565
categories:
- Miscellaneous Stuff
tags:
- assembly visualizer
- Reversing
---

[![ReverseEngineerBook](http://dandylife.net/blog/wp-content/uploads/2016/01/ReverseEngineerBook.png)](http://dandylife.net/blog/archives/565/reverseengineerbook)

[Dennis Yurichev](http://yurichev.com/) released an excellent reversing book for free, "[Reverse Engineering for Beginners](http://beginners.re/)". This allows readers who eager to learn reversing to do so systematically without worrying about affordability. The book is available in both English and Russian version, and full and (lite) introductory version. If you are looking for Korean version, it is also available thanks to 민병호 (translator) and Acorn Press - "[실전 연습으로 완성하는 리버싱](http://www.acornpub.co.kr/book/reversing-for-beginners)". All challenges/exercises/problems/tasks are accessible [here](http://challenges.re/).

With a lot of fruitful exercises, I found it very useful with the tool, _[C/C++ to assembly visualizer](https://github.com/ynh/cpp-to-assembly)_. Literally, it visualizes the relationship between C/C++ and assembly code via an awesome HTML web interface. It takes advantage of [Node.js](https://nodejs.org).

Here is a simple step to install the interface.
    
    $ sudo apt-get install npm
    ... (omitted) ...
    
    $ npm install -d
    ... (omitted) ...
    cluster@0.7.7 node_modules/cluster
    ├── log@1.4.0
    └── mkdirp@0.5.1 (minimist@0.0.8)
    
    express@3.0.0-rc4 node_modules/express
    ├── methods@0.0.1
    ├── fresh@0.1.0
    ├── range-parser@0.0.4
    ├── cookie@0.0.4
    ├── crc@0.2.0
    ├── commander@0.6.1
    ├── mkdirp@0.3.3
    ├── debug@2.2.0 (ms@0.7.1)
    ├── send@0.0.4 (mime@1.2.6)
    └── connect@2.4.4 (pause@0.0.1, bytes@0.1.0, qs@0.4.2, formidable@1.0.11)
    
    stylus@0.53.0 node_modules/stylus
    ├── css-parse@1.7.0
    ├── debug@2.2.0 (ms@0.7.1)
    ├── mkdirp@0.5.1 (minimist@0.0.8)
    ├── source-map@0.1.43 (amdefine@1.0.0)
    ├── glob@3.2.11 (inherits@2.0.1, minimatch@0.3.0)
    └── sax@0.5.8
    
    forever@0.15.1 node_modules/forever
    ├── path-is-absolute@1.0.0
    ├── object-assign@3.0.0
    ├── colors@0.6.2
    ├── clone@1.0.2
    ├── timespan@2.3.0
    ├── nssocket@0.5.3 (eventemitter2@0.4.14, lazy@1.0.11)
    ├── optimist@0.6.1 (wordwrap@0.0.3, minimist@0.0.10)
    ├── cliff@0.1.10 (eyes@0.1.8, colors@1.0.3)
    ├── winston@0.8.3 (cycle@1.0.3, stack-trace@0.0.9, eyes@0.1.8, isstream@0.1.2, async@0.2.10, pkginfo        @0.3.1)
    ├── shush@1.0.0 (strip-json-comments@0.1.3, caller@0.0.1)
    ├── utile@0.2.1 (deep-equal@1.0.1, ncp@0.4.2, async@0.2.10, i@0.3.4, mkdirp@0.5.1, rimraf@2.5.0)
    ├── prettyjson@1.1.3 (colors@1.1.2, minimist@1.2.0)
    ├── nconf@0.6.9 (ini@1.3.4, async@0.2.9, optimist@0.6.0)
    ├── forever-monitor@1.6.0 (minimatch@2.0.10, ps-tree@0.0.3, broadway@0.3.6, chokidar@1.4.2)
    └── flatiron@0.4.3 (optimist@0.6.0, director@1.2.7, prompt@0.2.14, broadway@0.3.6)
    npm info ok

If you have an error while starting _npm_ due to node stuff, creating symbolic might help. Once you start your own web server, check out the port 8080 open as following. 
    
    $ ln -s /usr/bin/nodejs /usr/bin/node
    $ sudo npm install coffee-script
    
    $ npm start
    > Assembly@0.0.1 start /root/gits/cpp-to-assembly
    > forever start -c coffee app.coffee
    warn:    --minUptime not set. Defaulting to: 1000ms
    warn:    --spinSleepTime not set. Your script will exit if it does not stay up for at least 1000ms
    info:    Forever processing file: app.coffee
    
    $ netstat -anlpt
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      27629/nodeNow you can enjoy&nbsp;a nice illustration to map your code to assembly. You might want to use&nbsp;<a href="https://assembly.ynh.io/" target="_blank" data-mce-href="https://assembly.ynh.io/">the demo</a> as well.

Now you can enjoy a beautiful illustration to map your code to assembly, by pasting your own code. You may want to check out the [demo](https://assembly.ynh.io/) in advance. The following example shows how the function _snprintf()_ is converted into machine-level code in a 64-bit machine.

[![code_comparison](http://dandylife.net/blog/wp-content/uploads/2016/01/code_comparison.png)](http://dandylife.net/blog/archives/565/code_comparison)
