---
layout: post
title: 'IoT Saga - Part 6 - Connecting Java and Spring to MQTT'
date:   2017-11-01 00:00:00
categories: java mqtt spring
disqus: true
description: Showing how to connect Java on top of Spring to MQTT
cover: /assets/images/bitfrost_logo_white.png
---

Hello, after a little gap, I'm trying to return to this project, many things are happening and I'm dealing with it as best as I can. So, let's start. The last post was about the main idea behind the Bitfrost concept, well, on this one I will show how to connect a Java project to MQTT using spring and some jars.

As I've said the Bitfrost project is responsible to connect Arduino and MQTT, then to reach this goal, this project should be able to send messages to MQTT dealing with Topics and the messages at all.

### Starting the Java project

At least for me, starting a java project is a time consuming task, we have to create many folders, dependencies files (Maven/Gradle) and follow some guidelines as the project package and so on. Unlike the Mix (Elixir) that gives to us a good bootstrap way to start, and the simplicity of the bundler with Gemfile (Ruby), I've always suffered to start to code. However, a friend of mine showed the [Spring Initializr][spring-initializr] some time ago. I know, I'm a little bit outdated about Java technologies but try to understand me ok? :).

#### References
* https://start.spring.io/

[spring-initializr]: https://start.spring.io/

