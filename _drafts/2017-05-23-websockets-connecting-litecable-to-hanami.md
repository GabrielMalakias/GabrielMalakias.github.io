---
layout: post
title: "IoT Saga - Part 3 - Websockets! Connecting LiteCable to Hanami"
date:   2017-05-23 00:00:00
categories: ruby hanami iot
disqus: true
description: Trying to show why and how I used websockets with Hanami
---

Hello, Today I wanna talk about how I use Websockets in Hanami. Well, I was worried about this line that I added in my application.rb

``` ruby
security.content_security_policy %{
connect-src: ws: 'self';
}
```
As I mentioned, I was using JS to connect to MQTT directly but I think isn't a good option because credentials can be exposed and I must control who is receiving and sending data through MQTT.

I started to search on Internet a good option when I suddenly saw this page:

![anycable demo]({{ site.url }}/assets/images/anycable_demo.png)

That sounds good then let`s use it!

### How it works

They believe that Ruby and Rails aren't the best option for websockets based on benchmarks, then they decided to extract the WebSocket responsability to another language, in this case, the language selected was Go.
Anycable-Go deals with the websocket management and many other things without know of any business logic.
To deal with this layer, we must create our classes to manage our rules, however how does AnyCable WebSocket(GO) connect to our Ruby Application?

They solved this problem using a gRPC client connected to another ruby process like the following picture.

![anycable demo]({{ site.url }}/assets/images/anycable_architecture.png)

* *Extracted from: https://evilmartians.com/chronicles/anycable-actioncable-on-steroids*

Well, They explain all pieces in this [post][anycable-on-steroids], please check it out.

I chose Hanami as Framework, I was looking for anyone that already made the connection between Hanami and Anycable but I didn't find anything. That's is reason why I decided to do it by myself and I will share my experience along this post. Fortunately Anycable already has a example using Sinatra, I basically followed these steps changing some pieces, let's move on.

### Adding pieces to Setup

Firstly we need a script to start our RPC server. I used the following code to start the Anycable RPC server and load Hanami dependencies.

``` ruby
require "rack"
require "anycable"
require_relative './config/boot'

LiteCable.anycable!

Anycable.configure do |config|
  config.connection_factory = Ws::Connection
end

Anycable::Server.start

```
This server is a rack application then we need to require rack and also the 'config/boot.rb' which will load all Hanami components using 'Hanami.boot'. After that the line 'LiteCable.anycable!' will enable the anycable compatibility mode. We must configure what is the class resposible to handle the connections, in this case 'Ws::Connection'. In the end the server must be started,  then 'Anycable::Server.start' do it.

In sinatra example they've shown how start anycable-go and the RPC server using hivemind to start all processes. I use docker-compose then I added the following line to my compose file.
``` yml
  rpc:
    build: .
    command: bundle exec ruby anycable
    volumes:
      - .:/usgard
    env_file:
      - .env.development
    environment:
      - ANYCABLE_REDIS_URL=redis://redis:6379/0
      - ANYCABLE_RPC_HOST=0.0.0.0:50051
    depends_on:
      - redis
      - db
  anycable:
    image: 'anycable/anycable-go:0.3'
    ports:
      - "8080:8080"
    environment:
      - ADDR=0.0.0.0:8080
      - REDIS=redis://redis:6379/0
      - RPC=rpc:50051
    depends_on:
      - redis
      - rpc
```

I run the RPC server using 'bundle exec anycable' and the Anycable-Go image will start automatically, We must only configure some environment variables, though.

*Ps:. The variable 'DATABASE_URL' must contains the connection string when 'Hanami.boot' is executed!*

Now, the basic infrastructure is prepared to handle all websocket connections.

### Creating Channels and Connections

### The JS

### References

* http://anycable.io
* https://evilmartians.com/chronicles/anycable-actioncable-on-steroids

[anycable-on-steroids]: https://evilmartians.com/chronicles/anycable-actioncable-on-steroids
