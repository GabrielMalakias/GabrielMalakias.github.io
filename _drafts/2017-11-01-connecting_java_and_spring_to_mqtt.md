---
layout: post
title: 'IoT Saga - Part 6 - Connecting Java and Spring to MQTT'
date:   2017-11-01 00:00:00
categories: java mqtt spring
disqus: true
description: Showing how to connect Java on top of Spring to MQTT
cover: /assets/images/bitfrost_logo_white.png
---





There is a diagram that explains the main idea:

![the bitfrost concept]({{ site.url }}/assets/images/the_bitfrost_concept.png)


### MQTT Broker

I'm using [mosquitto][mosquitto] as MQTT broker. This broker is capable to deal with the message's flux using channels, these channels are like routes or paths where we can put some information (messages). Take a look at the example below.

{% highlight shell %}
# Incomming message
MQTT Topic: /actuator/2/send

MQTT message headers: {mqtt_retained=false, mqtt_qos=0, id=78b2dcd1-21d1-9eb1-0bc1-c74c5d238e3b, mqtt_topic=actuator/2/send, mqtt_duplicate=false, timestamp=1502160412777}

MQTT message payload: on

{% endhighlight %}


It sounds familiar, isn’t it? Last week I’ve read a [post][mqtt-post] about MQTT and it gave me the basic knowledge to start to think about this application. For now I don’t want to be worried about permissions and abilities for a specific user because, at least for now, it’s only my hobby project. However I think it’s very easy to add this kind of functionality because I can use the user as a level for a channel like ‘user/123/sensor/1'

#### References
* http://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices
* http://rxtx.qbang.org/wiki/index.php/Installation_on_Linux
* https://mosquitto.org/

[mqtt-post]: http://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices
[mosquitto]: https://mosquitto.org/
