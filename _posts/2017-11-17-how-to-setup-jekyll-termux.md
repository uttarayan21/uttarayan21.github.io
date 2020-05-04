---
layout: post
title: How to setup Jekyll in Termux
title_color: grey
date: 2017-11-17 22:29:05 +0530
categories: android web termux jekyll
author: uttarayan21
published: true
image: /assets/images/jekyll.png
comments: true
excerpt: How to setup static site generator jekyll on android and update websites on the go 
---
### Jekyll on Termux

<br/>

~~Note that termux is  little buggy in android oreo so I had to make it a bit less inconvenient so that it would work in oreo~~
**NVM termux-api now has oreo support !!**

<br/>

I just made a script to make installation of jekyll easier in termux

With this you can upload your github hosted jekyll website on the go !


First install
[Termux](https://play.google.com/store/apps/details?id=com.termux)
 and
[Termux-api](https://play.google.com/store/apps/details?id=com.termux.api)
from PlayStore or
from
F-droid [termux](https://f-droid.org/en/packages/com.termux.api)
[Termux-Api](https://f-droid.org/en/packages/com.termux.api)


then install git in termux using

{% highlight bash %}
apt update && apt dist-upgrade

apt install git
{% endhighlight %}

then clone the repo

{% highlight bash %}
git clone https://github.com/uttarayan21/jekyll-termux
{% endhighlight %}

then enter the directory and start the script

{% highlight bash %}
cd jekyll-termux && ./setup.sh
{% endhighlight %}

and keep following the instructions
