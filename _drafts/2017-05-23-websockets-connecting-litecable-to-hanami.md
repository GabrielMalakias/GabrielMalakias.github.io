---
layout: post
title: "IoT Saga - Part 3 - Websockets! Connecting LiteCable to Hanami"
date:   2017-05-23 00:00:00
categories: ruby hanami iot
disqus: true
description: Trying to show why and how I used websockets with Hanami
---

Hello, Today I wanna talk about how I'm using Websockets in Hanami. Well, when I was starting I added the following line inside the application.rb but after that I was worried about it.

``` ruby
security.content_security_policy %{
connect-src: ws: 'self';
}
```
I was using PahoJS to connect to MQTT directly, on the other hand I think isn't a good option because credentials can be exposed and above all I must control who is receiving and sending data through MQTT in future.

I started to search on the Internet a good option when suddenly I saw this page:

![anycable demo]({{ site.url }}/assets/images/anycable_demo.png)

That sounds good, then why not?!

### How it works

Vladimir Dementyev([@palkan_tula][palkan]) recently wrote a post about it, From his point of view Ruby and Rails aren't the best option for websockets based his experience and benchmarks, a decision has been made, they(Anycable.io) decided to extract the WebSocket responsability to another language, in this case, the language selected was Go.
Anycable-Go deals with the websocket management and many other things without know of any business logic.
To deal with this layer, we must create our classes to manage our rules, however how does AnyCable WebSocket(GO) connect to a Ruby Application?

They solved this problem using a gRPC client connected to another ruby process like the following picture.

![anycable demo]({{ site.url }}/assets/images/anycable_architecture.png)

* *Extracted from: https://evilmartians.com/chronicles/anycable-actioncable-on-steroids*

Well, They explain all pieces in this [post][anycable-on-steroids], it's very interesting, please check it out.

I chose Hanami as Framework, I was looking for anyone that already made the connection between Hanami and Anycable but I didn't find anything. That's is reason why I decided to do it by myself and I will share my experience along this post. Fortunately Anycable already has a example using Sinatra, I basically followed these steps changing some pieces, let's start!

### Adding pieces to setup

Firstly, we need a script to start our RPC server. I used the following code to start the Anycable RPC server and load Hanami dependencies.

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
This server is a rack application then we need to require rack and also the 'config/boot.rb', which will load all Hanami components using 'Hanami.boot'. After that the line 'LiteCable.anycable!' will enable the anycable compatibility mode. After that we must configure what's the class resposible to handle the connections, in this case 'Ws::Connection'. In the end the server must be started, then 'Anycable::Server.start' do it.

In sinatra example they've shown how start anycable-go and the RPC server using hivemind to start all processes. I use docker-compose, then I added the following lines to my compose file.

``` yml
services:
  #More stuff here
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
* *Ps:. You can check this file [here][docker-compose]*

RPC server starts running 'bundle exec anycable' and the Anycable-Go image will start automatically, We must only configure some environment variables, though. Anycable uses Redis to manage the connections and broadcasts.

*Ps:. The variable 'DATABASE_URL' must contains the connection string when 'Hanami.boot' is executed!*

Now, the basic infrastructure is prepared to handle all websocket connections. Finally we can start to add the business logic.

### Creating Channels and Connections

LiteCable is a ActionCable implementation, I think Rails defines whole concepts behind it very well. The paragraph below has been extracted from Rails doc.

***For every WebSocket accepted by the server, a connection object is instantiated. [...] The connection itself does not deal with any specific application logic beyond authentication and authorization.*** - [Rails ActionCable Overview][rails-actioncable-overview]

We must create a class to deal with this layer.

``` ruby
class Connection < LiteCable::Connection::Base # :nodoc:
  identified_by :user, :sid

  def connect
    @user = 'usgard' #cookies["user"]
    @sid = request.params["sid"]
    reject_unauthorized_connection unless @user
    Hanami::Logger.new.info "#{@user} connected"
  end

  def disconnect
    Hanami::Logger.new.info "#{@user} disconnected"
  end
end
```

Rails defines channels as ***a logical unit of work, similar to what a controller does in a regular MVC setup.***


``` ruby
class Channel < LiteCable::Channel::Base # :nodoc:
  identifier :sensor

  def subscribed
    reject unless sensor_id
    stream_from "chat_#{chat_id}"
  end

  def speak(data)
    LiteCable.broadcast "chat_#{chat_id}", user: user, message: data["message"], sid: sid
  end

  private

  def chat_id
    params.fetch("id")
  end
end
```

### The JS

### References

* http://anycable.io
* https://evilmartians.com/chronicles/anycable-actioncable-on-steroids

[anycable-on-steroids]: https://evilmartians.com/chronicles/anycable-actioncable-on-steroids
[palkan]: https://twitter.com/palkan_tula
[docker-compose]: https://github.com/GabrielMalakias/usgard/blob/anycable_integration/docker-compose.yml
[rails-actioncable-overview]: http://guides.rubyonrails.org/action_cable_overview.html

