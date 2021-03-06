---
layout: post
status: publish
published: true
title: Bash Alias
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://github.com/cacqw7
date: '2015-01-12 02:27:59'
categories: cli
tags: mac web development code how-to learning programming learn-to-code coding osx dev environment wordpress-command-line bash-alias

---
# Make Bash (Command Line) Aliases
If you do even a little work on the command line (which you should), bash aliases are going to make your life infinitely easier.

With about three minutes of setup, you can save a huge amount of unnecessary typing.

### Edit `.bash_profile`

From your root directory, open `.bash_profile` with your favorite text editor.

{% highlight bash %}
$ sudo nano .bash_profile
{% endhighlight %}
### Add aliases

Create aliases using the syntax: `alias='command here'`
Example `.bash_profile`

{% highlight bash %}
if [ -f ~/.bashrc ];
then
   source ~/.bashrc
fi

#aliases
sites='cd ~/Sites'
..='cd ..'
...='cd ../ ..'
{% endhighlight %}

### Save and Reload shell
Save your file (Ctrl-O Ctrl-X for nano) and reload the shell by executing

{% highlight bash %}
$ source .bash_profile
{% endhighlight %}

### And that's it!
Happy coding!
