---
layout: post
title: "IoT Saga - My first setup for a hanami application"
date:   2017-01-31 12:00:00
categories: iot hanami docker
disqus: true
description: In this post, I will show what is a dependency injection how can we use this concept
---

Hello, Today I'm gonna talk about Hanami, I really wish to use it, them I decided to create an project to combine everything that I want to learn. I intend to do a saga covering my journey to complete it.

### Introduction

Basically I'm going to show how I made my setup to a new Hanami application. Hanami is an modular web frameworks that allows you to do applications decoupled based on [Clean Architeture][clean_architeture] and [Monolith First][monotith_first]. Docker turn easier to setup an run applications, it encapsulates an environment inside of a container to run a code I want. I'm going to use it too. A good tip gave by a colleague is [Scripts to rule them all][script_rule_them_all], it's good because you can use it as a convention to run projects in different languages and frameworks keeping on mind only the script concept for every project.

### Starting

##### 0. Creating an Hanami application
The first thing to do is create an application that we what to run, we can do it running this command to create an application:

{% highlight ruby %}

hanami new <project-name>

# In my case I used
hanami new space_wing --database=postgresql

{% endhighlight %}

*Ps:. You can pass options to it, for example database, test framework and etc, you can check all options at the [code][command_new]. or on [site][hanami]*

#### 1. Running on Docker

To run Docker first we need to install it, we will want docker and docker-compose, but I won't cover installation here to simplify this post, but you can check all proceds to install docker at the [site][docker].

Afterwards, we can run it to check docker version.

{% highlight shell %}
~/projects/space_wing(master ✔) docker --version
Docker version 1.12.1, build 23cf638

{% endhighlight %}

*Ps:. Currently it's stable version but you can use superior versions.*

We need an file called 'Dockerfile' to specify all steps to build our application image, so we will start creating it.

{% highlight shell %}
~/projects/space_wing(master ✔) touch Dockerfile
{% endhighlight %}



### References

* https://martinfowler.com/bliki/MonolithFirst.html
* https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
* https://github.com/github/scripts-to-rule-them-all
* https://github.com/hanami/hanami/blob/master/lib/hanami/commands/new/app.rb
* http://hanamirb.org

[hanami]: http://hanamirb.org
[command_new]: https://github.com/hanami/hanami/blob/master/lib/hanami/commands/new/app.rb
[monotith_first]: https://martinfowler.com/bliki/MonolithFirst.html
[clean_architeture]: https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
[script_rule_them_all]: https://github.com/github/scripts-to-rule-them-all
