---
layout: post
title: "IoT Saga - My first (to development) setup for a hanami application"
date:   2017-02-14 12:00:00
categories: hanami iot docker
disqus: true
description: I'm going to show how I made my setup basic development to a new Hanami application...
---

Hello, Today I'm gonna talk about Hanami, I really wish to use it, them I decided to create a project to combine everything that I want to learn. I intend to do a saga covering my journey to complete it. I intend to use some technologies like MQTT and RXTX to do the communication with my Arduino and after that an application to execute some commands on my board. This saga is inspired by [this post][mqtt-sergioaugrod] (made by one friend of mine), but I decided to use different tools, languages or frameworks to get some new knowledge.

The first post is about the high level application, responsible to send and receive all commands like a kind of dashboard where I can send commands, indicators and other things.

### Introduction

Basically, I'm going to show how I made my setup basic development to a new Hanami application, but first what's Hanami? My first definition is Hanami, based on website, is a modular web framework that allows you to do applications decoupled based on [Clean Architeture][clean_architeture] and [Monolith First][monotith_first].

Ok, now I have the framework, but I don't need to install all dependencies at my machine then I asked myself. *"How can I simplify my setup?"* One possibility is Docker. I decided to use it because I like :) and it turns easier to do the setup and to run applications. With Docker we can encapsulate all environment inside a container.

Another good tip, gave by a colleague, is [Scripts to rule them all][script_rule_them_all], it's convenient because following these rules we can use it like a convention to run projects in different languages and frameworks keeping on mind we need only to run a script inside a folder to test, run and build.

### Starting

##### 0. Creating an Hanami application
After the hanami installation (running 'gem install hanami'), the first thing to do is create an application that we want to run, we can do it running the command below.

{% highlight ruby %}

hanami new <project-name>

# In my case I used
hanami new space_wing --database=postgresql

{% endhighlight %}

*Ps:. You can pass options to it, for example database, test framework and etc, you can check all options at the [code][command_new]. or on [site][hanami]*

#### 1. Running on Docker

To run Docker first we need to install it, we want to install docker and docker-compose, but I won't cover installation here to simplify this post, but you can check do it at the [docker website][docker_install].

After installation, we can check it.

{% highlight shell %}
~/projects/space_wing(dev ✔) docker --version
Docker version 1.12.1, build 23cf638
{% endhighlight %}

*Ps:. Currently it's stable version but you can use superior versions.*

We need the file called 'Dockerfile' to specify all steps to build our application container, so we can start creating it.

{% highlight shell %}
~/projects/space_wing(dev ✔) touch Dockerfile
{% endhighlight %}

{% highlight shell %}
#Dockerfile content
FROM ruby:2.3.3

RUN mkdir /space_wing
WORKDIR /space_wing

ADD Gemfile .
ADD Gemfile.lock .

RUN bundle install

ADD . /space_wing

{% endhighlight %}

After that, we can use the command 'docker build .' to create a container with my application inside it. We also need to build all dependencies like the database and link between all containers, to do it we need to create the file called 'docker-compose.yml'. This file is responsible to build all dependencies and the network between all containers, then let's create it.

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

And the .env file to store our environment variables, in our case a database string connection.

{% highlight shell %}
~/projects/space_wing(dev ✔) touch .env

#.env content
DATABASE_URL=postgres://spacewing:inicial1234@db/spacewing_development

{% endhighlight %}

And when we run 'docker-compose up', we got it:

![welcome_to_hanami]({{ site.url }}/assets/images/welcome_to_hanami.png)

It's cool, right?

#### 2. Adding some scripts

In docker-compose we added this line:

{% highlight yml %}
command: bundle exec hanami s --host '0.0.0.0'
{% endhighlight %}

This command is responsible to run the app. If I'm at phoenix application for example we need to write a command like below.

{% highlight sh %}
command: mix phoenix.server
{% endhighlight %}

However we need to remember *"how can I run my server?", "how can I execute tests?", "how can I do all setup to my application?".* If I have some migrations, I need to create the database and run migrations.
Afterwards, I can forgot to initialize git submodules If I have any. I need to remember what command I need to run to install my dependencies and so on.

Now, imagine when we have a codebase with many languages, we can spend much time only remembering how do the setup for the application before start some task. From my point of view it's a caos.


One of possible solutions comes from [Script rule them all][script_rule_them_all]. It has a mission to do show some tips to automate some common tasks in your project, we can use it for any language or framework keeping on mind only the main idea behind that.


###### 2.1 Script to run tests

Let's create a folder to store all scripts, and then we need to add two scripts, one is responsible to install the dependencies and the another one is responsible to run our tests.

{% highlight shell %}
~/projects/space_wing(dev ✔) mkdir script
~/projects/space_wing(dev ✔) touch ./script/bootstrap
~/projects/space_wing(dev ✔) touch ./script/test

# bootstrap script content
#!/bin/sh

# script/bootstrap: Resolve all dependencies that the application requires to
#                   run.

echo "==> Building and solving dependencies…"

if command -v docker-compose >/dev/null 2>&1; then
    docker-compose build
else
    echo "==> Please install docker first"
fi

# test script content
#!/bin/sh
# script/test: Run test suite for application. Optionally pass in a path to an
#              individual test file to run a single test.

echo "==> Running tests…"

./script/bootstrap

if [ -n "$1" ]; then
  docker-compose run web rake test "$1"
else
  docker-compose run web rake test
fi
{% endhighlight %}

The script 'test' is resposible to run tests, to execute it, we need to run './script/test' or './script/test <file-test-path>' to run a single test.

###### 2.2 Script to run server

As the same example above, now, we can create a script to run the server. We can use the script below to execute it inside of container.

{% highlight shell %}

~/projects/space_wing(dev ✔) touch ./script/server

# server script content
#!/bin/sh
# script/server: Launch the application and any extra required processes
#                locally.
set -e
cd "$(dirname "$0")/.."

echo ">>> Starting application..."

if command -v docker-compose >/dev/null 2>&1; then
  docker-compose up
else
  echo "==> Please install docker first"
fi
{% endhighlight %}

We can add many other scripts to different proposes trying to remember the rules, I don't wanna to cover all scripts but you can check it at the project [page][script_rule_them_all].

That's all folks! We can add ci like travis or wercker but I'm going to stop here. I really wish you like it, and if you like it, please share with your friends.

### References

* https://martinfowler.com/bliki/MonolithFirst.html
* https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
* https://github.com/github/scripts-to-rule-them-all
* https://github.com/hanami/hanami/blob/master/lib/hanami/commands/new/app.rb
* http://hanamirb.org
* https://www.sergioaugrod.com.br/iot-arduino-serial-e-mqtt/
* https://docs.docker.com/engine/installation/

[mqtt-sergioaugrod]: https://www.sergioaugrod.com.br/iot-arduino-serial-e-mqtt/
[hanami]: http://hanamirb.org
[docker_install]: https://docs.docker.com/engine/installation/
[command_new]: https://github.com/hanami/hanami/blob/master/lib/hanami/commands/new/app.rb
[monotith_first]: https://martinfowler.com/bliki/MonolithFirst.html
[clean_architeture]: https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html
[script_rule_them_all]: https://github.com/github/scripts-to-rule-them-all
