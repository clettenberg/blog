---
layout: post
status: publish
published: true
title: Dokku + Ruby on Rails + Digital Ocean
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://github.com/cacqw7
date:   2015-05-28 15:54:14
categories: ruby-on-rails
---

### Using [Dokku](https://github.com/progrium/dokku) and [Digital Ocean](https://www.digitalocean.com/?refcode=e88ec69754d1) to create your own Heroku

[Dokku](https://github.com/progrium/dokku) is a PaaS built with [Docker](https://www.docker.com/) and designed to act like Heroku. You can easily deploy your apps to a remote production server from the command line at a fraction of the cost. You can even use subdomains to deploy multiple apps on the same Dokku install.

### Setup Rails Project
Create a new project
{% highlight bash %}
$ rails _4.1.8_ new PROJECTNAME
{% endhighlight %}
Add these lines to your Gemfile
{% highlight ruby %}
group :production do
 gem 'rails_12factor'
 gem 'pg'
 gem 'thin'
end
{% endhighlight %}
Then run
{% highlight bash %}
$ bundle update
$ bundle install
{% endhighlight %}
### Setup Dokku

You can install and run Dokku anywhere you'd like, but [Digital Ocean's](https://www.digitalocean.com/?refcode=e88ec69754d1) 1-Click Dokku install is too good to pass up.

* **Create and name your droplet**. I am currently running two apps on the smallest droplet, so any of them should work.
* Scroll down and work through the options until you get to **Select Image**. Click *Applications* and select *Dokku*.
* **Note** according to the [Dokku Docs](http://progrium.viewdocs.io/dokku/getting-started/install/digitalocean), you must disable IPv6 on your droplet.
* After the install finishes, **visit your droplets IP address** to finish the setup.
* Add your computer's ssh key.
* Select either to use the droplet's IP address or your custom domain.
* If you use a hostname, make sure to point your DNS to the droplet IP address for each Dokku app

### Deploying Your App
Setup git for your project
{% highlight bash %}
$ git init
$ git add -A
$ git commit -m "Init Commit"
{% endhighlight %}

Add the Dokku Remote

{% highlight bash %}
$ git remote add dokku dokku@example.com:projectname
$ git push dokku master
{% endhighlight %}

This will deploy your app to your droplet with the domain name **projectname.example.com**

###Setup PostgreSQL Database

SSH into your droplet and install the [Postgres Plugin](https://github.com/Kloadut/dokku-pg-plugin)

{% highlight bash %}
$ ssh root@example.com
$ cd /var/lib/dokku/plugins
$ git clone https://github.com/Kloadut/dokku-pg-plugin
$ dokku plugins-install
{% endhighlight %}

Create a Postgres container and link it to our project

{% highlight bash %}
$ dokku postgresql:create example-pg
$ dokku postgresql:link example example-pg
{% endhighlight %}

Migrate your database
{% highlight bash %}
$ dokku run example bundle exec rake db:migrate
{% endhighlight %}
#### Your site should be up and running! Repeat these steps for each project you put on your Dokku instance.

### Troubleshooting
Check logs with
{% highlight bash %}
$dokku logs example
{% endhighlight %}
