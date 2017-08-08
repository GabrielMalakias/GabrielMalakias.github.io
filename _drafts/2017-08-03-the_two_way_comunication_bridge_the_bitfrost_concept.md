---
layout: post
title: 'The two way communication bridge: Presenting the Bitfrost'
date:   2017-08-03 00:00:00
categories: java mqtt rxtx
disqus: true
description: In this post I will try to talk about circuit-breakers using ruby and stoplight gem
cover: /assets/images/bitfrost_logo_white.png
---

Hello, Today I will present what I’ve been thinking about this application. Well, first of all I have to explain the existence reason behind this application. Along the previous posts I’ve shown how I build the base to my dashboard panel using Hanami, Websockets and a MQTT broker, now it’s time to change the subject to start to talk about the application responsible for connect the Arduino world (Mygard) and the Dashboard world (Usgard). There is a diagram behind the main idea:

![the bitfrost concept]({{ site.url }}/assets/images/the_bitfrost_concept.png)

As you can see above the Bitfrost is a deamon application that runs threads to send and receive messages from MQTT and RXTX. There are many ways to make it possible, I thought that the best way is to use Convention over Configuration and just follow a guide to publish and subscribe many channels without configure all channels because I don’t wanna to change my application.yml everytime that I need to add a new sensor or an actuator.

The MQTT provides channels, these channels are like routes or paths where we can put some information about what’s the device that the message is comming. Take a look at the example below.

{% highlight shell %}
/sensor/1

/actuator/1
{% endhighlight %}


It sounds familiar, isn’t it? Last week I’ve read a [post][mqtt-post] about MQTT and it gave me the basic knowledge to start to think about this application. For now I don’t want to be worried about permissions and abilities for specific user because at least for now it’s only my hobby project but I think it’s very easy to add this kind of functionality because I can use the user as a level for a channel like ‘user/123/sensor/1'

I think that now the way that we deal with messages from MQTT, we have to think about the RXTX.

Thanks!

#### References
* http://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices

[mqtt-post]: http://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices
