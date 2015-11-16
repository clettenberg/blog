---
layout: post
status: publish
published: true
title: WordPress Command Line Interface
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://github.com/cacqw7
date: '2014-07-27 19:17:16 -0500'
categories: wordpress
tags: mac web-development tutorial codef how-to programming technology computers command-line-tools osx dev-environment wordpress blogging wordpress command-line
comments: []
---
If you use WordPress for your blog, you need to use WP-CLI (WordPress Command Line Interface). WP-CLI gives you a quick, easy, and customizable tool to keep your blog running smoothly. You can use the [vast amount of commands](http://wp-cli.org/commands/) that are already included, or make your own by just creating a new PHP class.

###Note: WP-CLI only works through SSH

**[Follow these instructions to install wp-cli](http://wp-cli.org/)**

Now, updating your WordPress version is as easy as

{% highlight bash %}
$ wp core update
{% endhighlight %}

You can also update your themes or plugins straight from the command line.

You can update individually or you can use “–all” to update the all at once.

For example, to update your ‘googleanalytics’ plugin, navigate to your plugins folder and type

{% highlight bash %}
$ wp plugin update googleanalytics
{% endhighlight %}

If you’d like to update all plugins in the directory type

{% highlight bash %}
$ wp plugin update --all
{% endhighlight %}

You can do the same with your themes, just replace ‘plugin’ with ‘theme’.

This is just scratching the surface of what WP-CLI is capable of doing. It will definitely make keeping up with endless updates on your WordPress a little less painful.

Again, [check their full list of commands](http://wp-cli.org/commands/) or create your own!
