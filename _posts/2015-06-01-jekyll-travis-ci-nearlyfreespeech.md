---
layout: post
status: publish
published: true
title: Static Blog with Jekyll and TravisCI
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
date:   2015-05-28 15:54:14
categories: ruby-on-rails
---

I finally took the time to ditch my old WordPress blog and start fresh with
a hip, new blogging engine powered by Markdown. Why make it easy on myself to
simply install a new blog and put it on some server, when I could decide to learn
continuous integration/deployment in the process.

*I made this blog using everything I'm about to describe, so feel free to jump straight into the [source](https://github.com/cacqw7/blog).*

**Tech Used**

 - [Jekyll](http://jekyllrb.com/) - Static Website Generator
 - [GitHub](http://github.com/) - Source Control
 - [Travis CI](https://travis-ci.org/) - Continuous Integration and Deployment
 - [Html Proofer](https://github.com/gjtorikian/html-proofer) - Validate Static files
 - [NearlyFreeSpeech](https://www.nearlyfreespeech.net/) - Web Host



### NearlyFreeSpeech
[NearlyFreeSpeech](http://nearlyfreespeech.net) is a great, low cost web host that I tend to use for any small scale personal projects. You can host a static site, like a Jekyll blog, for extremely cheap.

### Jekyll
[Jekyll](http://jekyllrb.com/) is a static website generator powered by Ruby. Customizable and easy to set up, supports Markdown and get's your blog into version control.

{% highlight bash %}
$ gem install jekyll
$ jekyll new myblog
$ cd myblog
$ ~/myblog $ jekyll serve
# Blog now served at localhost:4000
{% endhighlight %}

Jekyll has some great [docs](http://jekyllrb.com/docs/usage/) that can help you get your blog up and running.

### Travis CI and HTML-Proofer
[Travis CI](https://travis-ci.org/) is a continuous integration platform that makes it easy to build, test and deploy whenever changes are pushed to GitHub. The `cibuild` script will use html-proofer to check the `_site` folder for any errors, such as dead links.

Add `html-proofer` to the `Gemfile`
{% highlight ruby %}
gem "html-proofer"
{% endhighlight %}
Create a `.travis.yml` file

{% highlight yaml %}
language: ruby
rvm: 2.2.1
install: gem install jekyll html-proofer
script: jekyll build && ./script/cibuild
branches:
  #any branches you'd like Travis CI to test.
  only:
  - pages
  - master
  - "/pages-(.*)/"
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true

## before installing app
## Turn off strict host checking
## unzip encrypted ssh private key and _config.yml
before_install:
    - echo -e "Host ssh.phx.nearlyfreespeech.net\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
    - openssl aes-256-cbc -K $encrypted_dc941821cb90_key -iv $encrypted_dc941821cb90_iv
      -in secrets.tar.enc -out secrets.tar -d
    - tar xvf secrets.tar

after_success:
    - chmod 600 .travis/ssh_key
    - eval "$(ssh-agent -s)"
    - ssh-add .travis/ssh_key
    - rake deploy
{% endhighlight %}

Create a Ruby script at `script/cibuild`
{% highlight ruby %}
#!/usr/bin/env ruby

require 'html/proofer'
HTML::Proofer.new("./_site").run

{% endhighlight %}

I'm using `rsync` to deploy my `_site` folder to my [NearlyFreeSpeech](http://nearlyfreespeech.net) `/home/public/` folder.

Sample `Rakefile` file
{% highlight ruby %}
task :deploy do
  user = 'site_username'
  server = 'ssh.phx.nearlyfreespeech.net'
  sh "rsync -crz --delete _site/ #{user}@#{server}:/home/public"
end
{% endhighlight %}


To keep ssh keys and `Rakefile` settings out of the repo, zip them together and then [encrypt them](http://docs.travis-ci.com/user/encrypting-files/) so TravisCI can decrypt them.

First login to `travis`
{% highlight bash %}
$ travis login
{% endhighlight %}

Zip the files and encrypt them

{% highlight bash %}
# saves files .travis/ssh_key and Rakefile to secrets.tar
$ tar cvf secrets.tar Rakefile .travis/ssh_key
# encrypts and automatically adds it to .travis.yml
$ travis encrypt-file secrets.tar --add
{% endhighlight %}

**Make sure to add any encrypted files to `.gitignore`!**

### Recap!

* Create a blog with the `jekyll` command line constructer.
* Add blog content and add it to a git repo.
* Signup for [Travis CI](http://travis-ci.org) and enable the blog repo on [GitHub](https://github.com).
* Add a `.travis.yml` file
* Create a script for `html-proofer`
* Add `Rakefile` with NearlyFreeSpeech username
* Edit `_config.yml`
* [Create an ssh key](https://help.github.com/articles/generating-ssh-keys/) and add the public key to the NearlyFreeSpeech blog.
* Encrypt `.travis/private_key` and `Rakefile`
* Add and commit all the changes.
* Push the branch to GitHub and let Travis CI take care of testing and deploying.

Now you can brag about build passing like all the cook kids.

[![Build Status](https://travis-ci.org/cacqw7/blog.svg?branch=master)](https://travis-ci.org/cacqw7/blog)

But kidding aside, if you're at all new to continuous integration/deployment this is a great way to get your feet wet.
