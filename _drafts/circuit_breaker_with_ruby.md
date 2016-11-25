---
layout: post
categories: ruby circuit-breaker
---


At this year I went to QCon São Paulo, I saw many cool things and lots of new technologies. At the conference, some speeches was very until the end, others didn't has many interessed people. Some speakers talked about how we can make resilient systems. Kolton Andrus spoke a lot about of Netflix Toolset, he mentioned Eureka, Ribbon, Archaius and Hystrix. I liked so much of Hystrix, because the concept behind this mechanism sounds familiar and can be useful in many situations.

First, what's is Hystrix? The Netflix describes Hystrix as "Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.". Behind all advantages that it can give to us, has a lot of patterns but we will talk about only one, called of Circuit-Breaker.

First, we need to think about circuit-breaker. Let me see... Oh Circuit-Breaker pattern works like a home circuit-breaker, Obvious. But how it works? You can see it [here][homecircuitbreaker]. In IT world this mechanism can be applied to make a very useful behavior. Imagine that, you has a call to a external service, many things call fail, network, server down or something unexpected. If you wanna to make your system more resilient you can do some code improvements to prevent, and responds a default or a cache response if a service is down. When something goes wrong n times, the circuit-breaker(opened) stops to call the external service(or a failing method) and pass to responds only with a fallback method. After a time period, the circuit-breaker close and try to call a external service again, it can works or not. In this post i will try to show some examples with the same app at the previous post, you can download [experiences][Here].

I added Docker in this project only to turn easier the environment setup, you can use only ruby and rails to run.

I will talk only one gem available in Ruby, but if you search at RubyToolbox site you can found many similar solutions. The name of gem is [stoplight][Stoplight].


Ok how it works? Well, let's the see examples below! Ps: To run with docker you can use 'docker-compose up' and after that 'docker exec -it <image-name/id> bash'.

First thing to do is add gem in project, to do this we will add this line in Gemfile.

{% highlight ruby %}
gem 'stoplight'
{% endhighlight %}


In the folder of project we will type enter in rails console to do some experiments.

{% highlight bash %}
 ~/projects/ruby/blog  docker-compose up
 Building web
 Step 1 : FROM ruby:2.3.0
 ---> 7ca70eb2dfea
 Step 2 : RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
 ---> Using cache
 ---> 9f5de80da16a
...

...
 web_1                | * Version 3.6.0 (ruby 2.3.0-p0), codename: Sleepy Sunday Serenity
 web_1                | * Min threads: 5, max threads: 5
 web_1                | * Environment: development
 web_1                | * Listening on tcp://0.0.0.0:3000


~/projects/ruby/blog  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
68f52b94aed6        blog_web            "bash -c 'bundle exec"   5 minutes ago       Up 3 minutes        0.0.0.0:3000->3000/tcp   blog_web_1
7dfa02f6c2de        redis:3.0.7         "docker-entrypoint.sh"   8 minutes ago       Up 3 minutes        0.0.0.0:6379->6379/tcp   blog_experiences_redis_1


~/projects/ruby/blog  docker exec -it blob_web_1 bash
root@68f52b94aed6:/var/www/experiences# rails c
root@68f52b94aed6:/var/www/experiences# rails c
Running via Spring preloader in process 67
Loading development environment (Rails 5.0.0.1)

{% endhighlight %}

[homecircuitbreaker]: http://electronics.howstuffworks.com/circuit-breaker2.htm
[stoplight]: https://github.com/orgsync/stoplight
[experiences]: https://github.com/GabrielMalakias/experiences
