---
layout: post
title: "IoT Saga - My first (to development) setup for a hanami application"
date:   2017-01-31 12:00:00
categories: iot hanami docker
disqus: true
description: In this post, I will show what is a dependency injection how can we use this concept
---

Hello, Today I'm gonna talk about Hanami, I really wish to use it, them I decided to create an project to combine everything that I want to learn. I intend to do a saga covering my journey to complete it. I intend to use some technologies like MQTT and RXTX to do the communication with my Arduino and after that an application to execute some commands on my board, this saga is inspired by [it][mqtt-sergioaugrod], but I decided to use different tools, languages or frameworks to get some new knowledge.

The first post is about the high level application, responsible to send and receive all commands, I intend to make it like a dashboard where I can put commands, indicators and other things.

### Introduction

Basically, I'm going to show how I made my setup basic development to a new Hanami application, but first what's Hanami? My definition is Hanami is a modular web framework that allows you to do applications decoupled based on [Clean Architeture][clean_architeture] and [Monolith First][monotith_first].* Ok now I have an framework, but I don't need to install all dependencies at my machine then I asked myself. *"How can I simplify my setup?"* then I decided to use Docker because I like it :) and it turns easier to setup and run applications. With Docker we can encapsulate all environment inside a container.

A good tip gave by a colleague is [Scripts to rule them all][script_rule_them_all], it's good because you can use it as a convention to run projects in different languages and frameworks keeping on mind we need only to run a script inside a folder to test, run and build.

### Starting

##### 0. Creating an Hanami application
After the hanami installation (running 'gem install hanami'), the first thing to do is create an application that we want to run, we can do it running this command to create an application:

{% highlight ruby %}

hanami new <project-name>

# In my case I used
hanami new space_wing --database=postgresql

{% endhighlight %}

*Ps:. You can pass options to it, for example database, test framework and etc, you can check all options at the [code][command_new]. or on [site][hanami]*

#### 1. Running on Docker

To run Docker first we need to install it, we will want to install docker and docker-compose, but I won't cover installation here to simplify this post, but you can check all proceds to install docker and compose at the [docker website][docker].

After install, we need to check the docker version.

{% highlight shell %}
~/projects/space_wing(dev ✔) docker --version
Docker version 1.12.1, build 23cf638
{% endhighlight %}

*Ps:. Currently it's stable version but you can use superior versions.*

We need an file called 'Dockerfile' to specify all steps to build our application image, so we will start creating it.

{% highlight shell %}
~/projects/space_wing(dev ✔) touch Dockerfile

#Dockerfile content
FROM ruby:2.3.3

RUN mkdir /space_wing
WORKDIR /space_wing

ADD Gemfile .
ADD Gemfile.lock .

RUN bundle install

ADD . /space_wing

{% endhighlight %}

After that I can use the command 'docker build .' to create a image with my application inside it, so we need to build all dependencies like the database and link with our dockerized application, to do it we need to create a file called 'docker-compose.yml'. This file is responsible to build all dependencies and the network between all images. Then let's create it.

{% highlight shell %}
~/projects/space_wing(dev ✔) touch docker-compose.yml


{% endhighlight %}

{% highlight yml %}
#docker-compose.yml content
version: '2'

services:
  db:
    image: postgres
    environment:
      POSTGRES_DB: spacewing_development
      POSTGRES_USER: spacewing
      POSTGRES_PASSWORD: inicial1234
  web:
    build: .
    env_file:
      - .env
    command: bundle exec hanami s --host '0.0.0.0'
    volumes:
      - .:/space_wing
    ports:
      - "2300:2300"
    depends_on:
      - db

{% endhighlight %}

And the .env file to store my connection variable

{% highlight shell %}
~/projects/space_wing(dev ✔) touch .env

#.env content
DATABASE_URL=postgres://spacewing:inicial1234@db/spacewing_development

{% endhighlight %}

And then I got it:

![welcome_to_hanami]({{ site.url }}/assets/images/welcome_to_hanami.png)

Cool, right?

#### 2. Adding some scripts

In docker-compose I added this line:

{% highlight yml %}
command: bundle exec hanami s --host '0.0.0.0'
{% endhighlight %}

This command is responsible to run my app, If I'm at phoenix for example I need to run something like the line below.

{% highlight sh %}
mix phoenix.server
{% endhighlight %}

Then we need to remember *"how can I run my server?", "how can I execute tests?", "how can I do all setup to my application?".* If I have some migrations, I need to create the database and run migrations.
Afterwards, I can forgot to initialize git submodules If I have any. I need to remember what command I need to run to install my dependencies and so on.

Now, imagine when you have a codebase with many languages, You can spend a lots of time only remembering how do the setup for the application before start some task. It's a caos.

This github project has a mission to do some tips to automate some common tasks in your project, we can use it for any language or framework following only the idea of each one script.



### References

* https://martinfowler.com/bliki/MonolithFirst.html
* https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
* https://github.com/github/scripts-to-rule-them-all
* https://github.com/hanami/hanami/blob/master/lib/hanami/commands/new/app.rb
* http://hanamirb.org
* https://www.sergioaugrod.com.br/iot-arduino-serial-e-mqtt/


[mqtt-sergioaugrod]: https://www.sergioaugrod.com.br/iot-arduino-serial-e-mqtt/
[hanami]: http://hanamirb.org
[command_new]: https://github.com/hanami/hanami/blob/master/lib/hanami/commands/new/app.rb
[monotith_first]: https://martinfowler.com/bliki/MonolithFirst.html
[clean_architeture]: https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
[script_rule_them_all]: https://github.com/github/scripts-to-rule-them-all
