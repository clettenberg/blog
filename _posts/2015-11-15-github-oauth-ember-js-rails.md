---
layout: post
status: publish
published: true
title: GitHub OAuth with Ember.js and Rails 5 API
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
date:   2015-11-15 15:54:14
categories: ember-js
tags: ember-js ruby-on-rails api torii oauth github
---

For a recent [Ember.js](http://emberjs.com) + Rails API project, I needed to set up OAuth with support for [GitHub](https://developer.github.com/v3/oauth/). I used the [Torii](https://github.com/Vestorly/torii) library along with EmberCLI. This a rough, not quite finished version.

I'm using

- [EmberCLI](http://www.ember-cli.com/) v.1.13.8
- Ember.js v.2.1.0
- [Torii](https://github.com/Vestorly/torii) v.0.6.1
- Ruby on Rails API v.5

_**Before you get started**_

*Make sure your app is registered on your [GitHub](http://github.com) account.*

*Go to `Settings > Applications > Developer Applications`*

*To get things up and running in development, just set `Homepage URL` and `Authorization callback URL` to
`http://localhost:4200`*


### **Ember.js**

*I want to authenticate users who try to visit a page of shared links. Any code references to `links` can be interchanged with whichever model you are using in your project.*

Add [Torii](https://github.com/Vestorly/torii) to your `package.json`

{% highlight javascript %}
// package.json
"torii": "^0.6.1"
{% endhighlight %}

Configure `torii` in your `ENV` variable

{% highlight javascript %}
// config/environment.js
module.exports = function(environment) {
  var ENV = {
    torii: {
         sessionServiceName: 'session',
         providers: {
           'github-oauth2': {
             //Key GitHub gives you once you register your app
             apiKey: 'API_KEY_FROM_GITHUB',
             //scope for your authentication
             scope: 'user',
             //This is the link that it redirects you back to
             redirectURI: '/links'
           }
         }
       }
   ;}
 };
{% endhighlight %}

Create a new folder called `torii-adapters`

{% highlight javascript %}
// app/torii-adapters/application.js
export default Ember.Object.extend({
  open: function(authentication){
    var authorizationCode = authentication.authorizationCode;
    localStorage.token = authorizationCode;
    return new Ember.RSVP.Promise(function(resolve, reject){
      Ember.$.ajax({
        // url on your server for storing your session info
        url: 'http://localhost:3000/session',
        data: { 'access_code': authorizationCode },
        dataType: 'json',
        success: Ember.run.bind(null, resolve),
        error: Ember.run.bind(null, reject)
      });
    }).then(function(user){
      // The returned object is merged onto the session (basically). Here
      // you may also want to persist the new session with cookies or via
      // localStorage.
      return {
        currentUser: user.user
      };
    });
  },
  fetch: function() {
    if (!localStorage.token) {
      return rejectPromise();
    }
    return new Ember.RSVP.Promise(function(resolve, reject){
      Ember.$.ajax({
        url: 'http://localhost:3000/session/fetch',
        data: { 'access_code': localStorage.token },
        dataType: 'json',
        success: Ember.run.bind(null, resolve),
        error: Ember.run.bind(null, reject)
      });
    }).then(function(user){
      // The returned object is merged onto the session (basically). Here
      // you may also want to persist the new session with cookies or via
      // localStorage.
      return {
        currentUser: user.user
      };
    });
  }
});
{% endhighlight %}

Add a `login` route to your `router.js` file

{% highlight javascript %}
// app/router.js
Router.map(function(){
  this.route('login');
});
export default Router;
{% endhighlight %}

You need to create the `before_model` and `logout` hooks in your **application** route

{% highlight javascript %}
// app/routes/application.js
export default Ember.Route.extend({
  model: function(){
    return this.store.findAll('link');
  },

  beforeModel: function() {

    return this.get('session').fetch().then(function() {
      console.log('session fetched');
    }, function() {
      console.log('no session to fetch');
    });
  },

  actions: {
    logout: function() {
      this.get('session').close();
    }
  }
});
{% endhighlight %}

Create a new **route** for `login`

{% highlight javascript %}
// app/routes/login.js
export default Ember.Route.extend({
  actions: {
    signInViaGithub: function(){
      var route = this,
          controller = this.controllerFor('login');
      // The provider name is passed to `open`

      this.get('session').open('github-oauth2').then(function(){
        route.transitionTo('links');
      }, function(error){
        controller.set('error', 'Could not sign you in: '+error.message);
      });
    }
  }
});
{% endhighlight %}

Create a new `login` controller
{% highlight javascript %}
// app/controllers/login.js
import Ember from 'ember';

export default Ember.Controller.extend({

});
{% endhighlight %}

Template for logging in

{% highlight handlebars %}
{% raw %}
{{! app/templates/login.hbs}}
{{#if session.isWorking}}
  One sec while we get you signed in...
{{else}}
  {{error}}
  <a href="#" {{action 'signInViaGithub'}}>
    Sign In with Github
  </a>
{{/if}}
{% endraw %}
{% endhighlight %}

Login page

{% highlight javascript %}
// app/routes/link.js
import Ember from 'ember';

export default Ember.Route.extend({
  model: function(){
    return this.store.findAll('link');
  }
});
{% endhighlight %}

Toriii has a helper `authenticatedRoute`. I set it up for my `links` pages in my `router.js` file.

{% highlight javascript %}
// app/router.js
Router.map(function() {
  this.authenticatedRoute('links', function(){
    this.route('new');
  });

  this.route('login');

});

export default Router;
{% endhighlight %}

### Rails 5 API setup

I setup a `User` model in my Rails application. Here's what my migration file looks like

{% highlight ruby %}
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :login
      t.string :avatar_url
      t.string :github_url
      t.string :name
      t.string :company
      t.string :location
      t.string :email
      t.text :bio
      t.string :code
      t.timestamps
    end
  end
end
{% endhighlight %}

As I mentioned before, I'm using a `Link` feature. I needed three different urls to keep track of the sessions. This is my `routes.rb` file

{% highlight ruby%}
Rails.application.routes.draw do
  resources :links
  resources :users

  get "/session", to: "session#index"
  get "/session/fetch", to: "session#fetch"
  post "/session/logout", to: "session#logout"

end
{% endhighlight %}

To keep my sessions controller "skinny", I extracted my GitHub API logic into an [interactor](https://gist.github.com/cacqw7/9e8dd6139724fad890f1). After that, this is my `sessions_controller.rb`
{% highlight ruby %}
class SessionController < ApplicationController

  def index
    user = GithubApi.new(params[:access_code]).get_user
    render json: user
  end

  def fetch
    user = User.find_by(code: params[:access_code])
    render json: user
  end

  def logout
    user = User.find_by(code: params[:access_code])
    user.code = "0"
  end

end
{% endhighlight %}

With that, you should have all the pieces you need to get basic OAuth2 functionality with GitHub.
