---
layout: post
title: "Cài đặt Rails trên Ubuntu 14.04"
modified: 2015-07-28T14:02:19+07:00
comments: true
categories: Setup
excerpt: "Từng bước cấu hình môi trường để có thể phát triển Rails
project"
tags: ['Ubuntu', 'Ruby', 'Ruby on Rails']
image:
  feature:
date: 2015-04-29T12:59:26+07:00
---

## Cài đặt Ruby

### Cài đặt dependencies cho Ruby:

{% highlight sh %}
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential \
  libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 \
  libxml2-dev libxslt1-dev libcurl4-openssl-dev \
  python-software-properties libffi-dev
{% endhighlight %}

### Cài đặt Ruby qua rvm

{% highlight sh %}
sudo apt-get install libgdbm-dev libncurses5-dev \
  automake libtool bison libffi-dev
curl -L https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
rvm install 2.2.1
rvm use 2.2.1 --default
ruby -v
{% endhighlight %}

*Chú ý*: Nếu có đổi shell thì cần thêm vào zsh/bash config file:

{% highlight sh %}
export PATH="$PATH:$HOME/.rvm/bin" # Add RVM to PATH for scripting
{% endhighlight %}

Không cho Rubygems cài đặt docs cho package locally

{% highlight sh %}
echo "gem: --no-ri --no-rdoc" > ~/.gemrc
{% endhighlight %}

Cài đặt Bundler

{% highlight sh %}
gem install bundler
{% endhighlight %}

## Cài đặt Rails

Rails cần Javascript runtime, vậy nên cài thêm NodeJS:

{% highlight sh %}
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
{% endhighlight %}

Sau đó cài đặt rails:

{% highlight sh %}
gem install rails -v 4.2.0
{% endhighlight %}

Để kiểm tra rails đã được cài đặt thành công hay chưa, chạy:

{% highlight sh %}
rails -v
# Rails 4.2.0
{% endhighlight %}

## Cài đặt MySQL
Chỉ cần chạy:

{% highlight sh %}
sudo apt-get install mysql-server mysql-client libmysqlclient-dev
{% endhighlight %}

*Chú ý*: Nếu cần cài MySQL 5.6:

 - Gỡ bỏ MySQL 5.5

{% highlight sh %}
sudo apt-get purge mysql-server-5.5 mysql-client-5.5
sudo apt-get autoremove
{% endhighlight %}
 - Cài đặt MySQL 5.6

{% highlight sh %}
sudo apt-get install mysql-server-5.6 mysql-client-5.6
{% endhighlight %}

[source](https://gorails.com/setup/ubuntu/14.04)
