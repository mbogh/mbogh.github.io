---
layout: post
title: "Use RVM with Octopress on OSX Yosemite"
date: 2014-10-11 23:41:08 +0200
comments: false
categories: [Octopress,RVM,OSX,Yosemite,Ruby]
permalink: /:year/:month/:day/:title/
---

When the OS X Yosemite beta was released, I did a clean install of it on my MacBook Air.
Time went by and then I had time to blog again, so I did the following:

Firstly cloned the repository from Github.
{% highlight bash %}
git clone git@github.com:mbogh/mbogh.github.io.git
cd mbogh.github.io
{% endhighlight %}

Next up is installing `rbenv`.
{% highlight bash %}
brew update
brew install rbenv
brew install ruby-build
rbenv install 1.9.3-p0
rbenv local 1.9.3-p0
rbenv rehash
{% endhighlight %}

Lastly install the gem `bundler` and install missing gems.
{% highlight bash %}
gem install bundler
rbenv rehash
bundle install
rake install
{% endhighlight %}

**...booooom...**

Then I ranted to my ruby-capable friend Markus, he laughed and said always use `RVM` and so I did.

{% highlight bash %}
\curl -sSL https://get.rvm.io | bash -s stable
rvm install 2.1-head
rvm use --default 2.1-head
gem install bundler
bundle install
rake install
{% endhighlight %}

And now you are seeing this post :)
