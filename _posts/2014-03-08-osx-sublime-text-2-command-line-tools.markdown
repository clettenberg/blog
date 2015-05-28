---
layout: post
status: publish
published: true
title: OSX Sublime Text 2 Command Line Tools
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://www.cclettenberg.com
date: '2014-03-08 11:57:54 -0600'
categories:
- mac
- tutorial
- resources
- dev environment
tags:
- mac
- code
- programming
- sublime text 2
- subl
- command line tools
- text editor
- osx
- dev environment
- sublime text 2 command line tools
comments: []
---
As if you needed another reason to love [Sublime Text 2…](http://www.sublimetext.com/2) 

The easy to install [command line tools](https://www.sublimetext.com/docs/2/osx_command_line.html) make opening files from the command line a breeze.

Create a `subl` directory in your `bin` folder
	
	$ mkdir ~/[path_to_bin]/bin/subl

Install the command line tools (Assuming Sublime Text is installed in your **Applications** folder.
	
	$ ln -s "/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl" ~/[path_to_bin]/bin/subl
	
Now you’re all set!  Test it out by opening your current directory
	
	$ subl .
	
Head on over to [Sublime Text’s docs](https://www.sublimetext.com/docs/2/osx_command_line.html) to see a full list of available commands.
