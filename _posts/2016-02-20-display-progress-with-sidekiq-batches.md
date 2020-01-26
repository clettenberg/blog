---
layout: post
title: Display Progress with Sidekiq Batches
date: 2016-02-20
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
categories: ruby-on-rails
tags: sidekiq ruby-on-rails ruby background-processes sidkiq-pro sidekiq-batches
---

###### *Batches are only available in [Sidekiq Pro](http://sidekiq.org/products/pro)*

[Church360º Members](http://www.concordiatechnology.org/church360/members) lets users send end-of-year statement emails to their entire congregation. Some of our users could send be sending upwards of 3,000 emails at one time. We wanted a way to give the user a progress bar for the jobs instead of just notifying them once they are all sent.

End result using [Sidekiq Batches](https://github.com/mperham/sidekiq/wiki/Batches) and [progressbar.js](https://github.com/kimmobrunfeldt/progressbar.js):

![progress gif](/assets/img/progress.gif){: .post-img }

#### Batches

From the [docs](https://github.com/mperham/sidekiq/wiki), a batch is

> a collection of jobs which can be monitored as a group. You can create a set of jobs to execute in parallel and then execute a callback when all the jobs are finished.

This is our current code for creating jobs:
{% highlight ruby %}
selected_people.pluck(:id).each do |person_id|
  EmailWorker.perform_async(current_user.email, @start_date, @end_date, options)
end
{% endhighlight %}

Turning this into a batch is as simple as:
{% highlight ruby %}
batch = Sidekiq::Batch.new
batch.jobs do
  selected_people.pluck(:id).each do |person_id|
    EmailWorker.perform_async(current_user.email, @start_date, @end_date, options)
  end
end
{% endhighlight %}

Batches come with two types of [callbacks](https://github.com/mperham/sidekiq/wiki/Batches#callbacks):

- **success** - when all jobs in the batch have completed successfully.
- **complete** - when all jobs in the batch have run once, successful or not.

We enable inline mode in `sidekiq.rb` to avoid having to run [Redis](http://redis.io/) and a Sidekiq process at all times in development.

> inline mode runs the job immediately instead of enqueuing it

{% highlight ruby %}
# sidekiq.rb
if Rails.env.development?
  require 'sidekiq/testing'
  Sidekiq::Testing.inline!
end
{% endhighlight %}

Unfortunately, batch callbacks have to be called manually in [test modes](https://github.com/mperham/sidekiq/wiki/Testing#testing-batches). Instead of creating a bunch of conditionals, we can wrap our previous code so batches are created when Sidekiq is running and run immediately in inline mode.

{% highlight ruby %}
in_batch do
  selected_people.pluck(:id).each do |person_id|
    EmailWorker.perform_async(current_user.email, @start_date, @end_date, options)
  end
end

def in_batch
  return yield if defined?(Sidekiq::Testing)
  batch = Sidekiq::Batch.new
  batch.on(:success, BatchSuccessHandler)
  batch.jobs do
    yield
  end
end
{% endhighlight %}

Optionally add an environment variable to easily toggle inline mode.

{% highlight ruby %}
# sidekiq.rb
if Rails.env.development?
  unless ENV["ENABLE_SIDEKIQ"]
    require 'sidekiq/testing'
    Sidekiq::Testing.inline!
  end
end
{% endhighlight %}


Now you can have your jobs run with a Sidekiq instance if you start your server with `ENABLE_SIDEKIQ=1 bundle exec rails s`.

#### Progress Bar

The other half is giving the user a progress bar. Create a route that points to either a controller or interactor that can look up [batch status](https://github.com/mperham/sidekiq/wiki/Batches#status). Could be something similar to `/api/batch/status/:bid`. The important part is to return percentage finished to the browser.
{% highlight ruby %}
def progress
  status = Sidekiq::Batch::Status.new(bid)
  unless status.nil?
    unsent = status.pending + status.failures
    percent_complete = percent_complete(status.total, unsent)
  end
end
{% endhighlight %}

Our app has a `Message` model that the browser polls for every minute. Polling ramps up to every 3 seconds when any batches are in progress.

[progressbar.js](https://github.com/kimmobrunfeldt/progressbar.js) is an easy, lightweight way to create progress bars in javascript.

Here is the piece of the `coffescript` file that updates progress:

{% highlight coffeescript %}
# Create an array to keep track of multiple batch progress bars
progressCircles = []

# Initializes new progress circles
initializeProgress: (id, progressPercent) ->
 progressCircles[id] = new ProgressBar.Circle("#progress-circle-#{id}",
   color: "#83c7c7"
   strokeWidth: 3
   trailWidth: 1
   duration: 1500
   text: value: '0'
   step: (state, bar) ->
     bar.setText (bar.value() * 100).toFixed(0)
 )

# Updates or creates progress circles with new percentage
updateProgress: (id, progressPercent) ->
  initializeProgress(id, progressPercent) unless @progressCircles[id]?
  progressCircles[id].animate (progressPercent / 100)
{% endhighlight %}
---

This post is a quick snapshot of how we use batches to improve our customer's experience when sending emails. This just scratches the surface of what's possible.


Make sure to check out all the features of [Sidekiq](https://github.com/mperham/sidekiq/wiki) and [progressbar.js](https://github.com/kimmobrunfeldt/progressbar.js).
