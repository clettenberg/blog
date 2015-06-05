---
layout: post
status: publish
published: true
title: Bow Down to Markdown
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
date:   2015-05-28 15:54:14
categories: ruby-on-rails nearlyfreespeech jekyll blog travis-ci
---

I finally took the time to ditch my old WordPress blog and start fresh with
a hip, new blogging engine powered by Markdown. Why make it easy on myself to
simply install a new blog and put it on some server, when I could decide to learn
continuous integration/deployment in the process.

**Tech Used**

 - [Jekyll](http://jekyllrb.com/) - Static Website Generator
 - [GitHub](http://github.com/) - Source Control
 - [Travis CI](https://travis-ci.org/) - Continuous Integration and Deployment
 - [Html Proofer](https://github.com/gjtorikian/html-proofer) - Validate Static files
 - [NearlyFreeSpeech](https://www.nearlyfreespeech.net/) - Web Host

###NearlyFreeSpeech
[NearlyFreeSpeech](http://nearlyfreespeech.net) is a great, low cost web host that I tend to use for any small scale personal projects. You can host a static site, like a Jekyll blog, for extremely cheap. 

###Jekyll
[Jekyll](http://jekyllrb.com/) is a static website generator powered by Ruby. Customizable and easy to set up, supports Markdown and get's your blog into version control.

    $ gem install jekyll
    $ jekyll new myblog
    $ cd myblog
    ~/myblog $ jekyll serve
    # Blog now served at localhost:4000

Jekyll has some great [docs](http://jekyllrb.com/docs/usage/) that can help you get your blog up and running.

###Travis CI and HTML-Proofer
[Travis CI](https://travis-ci.org/) is a continuous integration platform that allows us to build, test and deploy our blog automatically whenever changes are pushed to GitHub. 

Travis CI is going to run html-prooferto check your _site folder for any errors such as dead links. 

Add `html-proofer` to your gemfile
		
	gem "html-proofer"

Create a `.travis.yml` file

	language: ruby
	rvm: 2.1
	install: gem install jekyll html-proofer
	script: jekyll build && htmlproof ./site
	branches: 
	  #any branches you'd like Travis CI to test. 
	  only:
	  - pages
	  - master
	  - "/pages-(.*)/"
	env:
	  global:
	  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true

	##before installing app
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

I'm using `rsync` to deploy my `_site` folder to my [NearlyFreeSpeech](http://nearlyfreespeech.net) `/home/public/` folder. 

Sample `Rakefile` file

	task :deploy do
	  user = 'site_username'
	  server = 'ssh.phx.nearlyfreespeech.net'
	  sh "rsync -crz --delete _site/ #{user}@#{server}:/home/public"
	end



To keep your ssh key and Rakefile settings out of the repo, you need to zip them together and then [encrypt them so TravisCI](http://docs.travis-ci.com/user/encrypting-files/) can unencrypt them. 

First login to `travis`

	$ travis login

Zip the files and encrypt them

	#saves files .travis/ssh_key and Rakefile to secrets.tar
	$ tar cvf secrets.tar Rakefile .travis/ssh_key 
	#encrypts and automatically adds it to .travis.yml
	$ travis encrypt-file secrets.tar --add 

**Make sure to add any encrypted files to your .gitignore!**

###Recap!

* Create your blog through the `jekyll` command line constructer. 
* Add blog content and add it to a git repo. 
* Signup for [Travis CI](http://travis-ci.org) and enable your blog repo. 
* Add a .travis.yml file
* Add Rakefile with nearly free speech username
* Edit _config.yml
* [Create an ssh key](https://help.github.com/articles/generating-ssh-keys/) and add the public key to your NearlyFreeSpeech blog.
* Encrypt `.travis/private_key` and `Rakefile`
* Add and commit all your changes.
* Push your branch to GitHub and let Travis CI take care of testing and deploying.

Then you can brag about build passing like all the cook kids. 

[![Build Status](https://travis-ci.org/cacqw7/blog.svg?branch=master)](https://travis-ci.org/cacqw7/blog)

But kidding aside, if you're at all new to continuous integration/deployment this is a great way to get your feet wet. 

