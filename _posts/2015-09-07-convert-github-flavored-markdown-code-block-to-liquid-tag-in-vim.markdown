---
layout: post
title: "Convert Github Flavored Markdown Code Block to Liquid Tag in VIM"
modified:
categories: vim
excerpt:
tags: [vim]
image:
  feature:
date: 2015-09-07T11:00:44+07:00
---

{% highlight text %}
{% raw %}
:%s/\v^```(.*)$(\_.{-})^```$/{% highlight \1 %}\2{% endhighlight %}/g | %s/{% highlight  %}/{% highlight text %}/g
{% endraw %}
{% endhighlight %}
