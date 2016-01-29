---
layout: post
title: Customized Git Log
date: 2016-01-07
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
categories: git
tags: git cli terminal dev-tools productivity
---

`git log` is a great tool to see the commit history of a project. The default output can be a little noisy and not exactly helpful.

{% highlight terminal %}
$ git log
commit 6a789bcaf223334d1e34bf220812869ebbeca853
Author: Chase Clettenberg <commiter@example.com>
Date:   Thu Jan 7 22:04:09 2016 -0600

    Update Ruby to 2.3.0 in travis.yml

commit 69fcbe8d4d946271da3463d3bdc0985adb8bd445
Author: Chase Clettenberg <commiter@example.com>
Date:   Thu Jan 7 22:01:35 2016 -0600

    Update to Ruby 2.3.0
...
{% endhighlight %}

Unsurprisingly, there are plenty of formatting options in the [documentation](http://git-scm.com/docs/git-log).

## Here are a few aliases that I've found useful:


#### **Branch and commit message**

`alias gl='git log --decorate --pretty=oneline --abbrev-commit'`
{% highlight terminal %}
$ gl
6a789bc (HEAD -> beautify-git-log, origin/master, master) Update Ruby to 2.3.0 in travis.yml
69fcbe8 Update to Ruby 2.3.0
0498a62 (remove) Remove executable
...
{% endhighlight %}

#### **Committer, when it was committed and commit message**

`alias gla='git log --pretty=format:"%C(yellow)%h%C(reset) - %an [%C(green)%ar%C(reset)] %s"'`
{% highlight terminal %}
$ gla
6a789bc - Chase Clettenberg [3 weeks ago] Update Ruby to 2.3.0 in travis.yml
69fcbe8 - Chase Clettenberg [3 weeks ago] Update to Ruby 2.3.0
0498a62 - Chase Clettenberg [3 weeks ago] Remove executable
...
{% endhighlight %}

#### **`grep` to search the logs for keywords**
`alias glog='git log -E -i --grep'`

{% highlight terminal %}
$glog link
commit 6c10b4e3b1e6d8289f496a0bed5668cac4c379ea
Author: Chase Clettenberg <commiter@mail.com>
Date:   Thu Jun 4 22:21:48 2015 -0500

    add github link

commit 423f6f7156722128f874c098b193bfc0c8d51459
Author: Chase Clettenberg <commiter@mail.com>
Date:   Sat May 30 17:02:10 2015 -0500

    update link
...
{% endhighlight %}
