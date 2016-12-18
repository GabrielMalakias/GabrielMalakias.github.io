---
layout: post
title: 'Playing with circuit-breakers'
date:   2016-12-08 00:00:00
categories: ruby circuit-breaker
disqus: true
description: In this post I will try to talk about circuit-breakers using ruby and stoplight gem
---


At this year I went to QCon São Paulo, I saw many cool things and lots of new technologies. In the conference, some speakers talked about how we can make resilient systems. Kolton Andrus ([slides][kolton-andrus]) spoke about of Netflix Toolset, he mentioned Spinnaker, Chaos Kong and Hystrix. I liked so much of Hystrix, because the concept behind this mechanism sounds very simple and can be useful in many situations. In this post, I will try to talk about it.

Oh, what is Hystrix? The Netflix describes Hystrix as "[...] a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.". Behind all advantages that it can give to us, it has a lot of patterns, but in this post I will write about only one, called of Circuit-Breaker.

We need to think about circuit-breaker. Let me see... Oh Circuit-Breaker pattern works like a home circuit-breaker, Obvious. But how it works? You can see it [here][homecircuitbreaker]. In IT world, this mechanism can be applied to some cases to prevent errors or decrease latency. Imagine, you have a call to an external service, in this scenario many things can fail, you can get network error, some server can be out, or something unexpected can occurs. If you want to make your system more resilient, you need to assume that your system can fail. To avoid that problem, you can respond a default response or with cache. Using a circuit-breaker, when something goes wrong N times, the circuit-breaker stops to call the failed method and pass to responds only with a fallback method. After a time period, the circuit-breaker try to call a method again, it can works or not.

In this post, I will try to show some examples with the same app at the previous post. You can download it [here][experiences](please use version 0.0.2). I added Docker only to turn easier the environment setup, but you can use only ruby(>=2.2) and rails(>=5) to run.

I will show only one circuit-breaker gem available in Ruby, but if you search at RubyToolbox site you can found many similar solutions like [circuitbox][circuitbox] or [resilient][resilient]. In this post we will use the [stoplight][Stoplight] gem.

Ok, how it works? Well, we will see it together along this post. It's easier, at least for me, understand with examples.

*Ps: To run with docker you can use 'docker-compose up' and after that 'docker exec -it <image-name/id> bash' to enter in bash. After that we need to type 'rails c'.*

First thing that we need to do is add a new gem in this project, to do it we need to add this line in our Gemfile.

{% highlight ruby %}
gem 'stoplight'
{% endhighlight %}

*Ps: Everytime that we add a new gem in Gemfile, we need to run 'docker-compose build' again and after that we can run 'docker-compose up'.*

In the project folder, we need to type the command below to start our server.

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

{% endhighlight %}

The command above build the image from Dockerfile, starts all dependencies that we added in docker-compose and executes a command specified in docker-compose (in our case 'bundle exec puma').

{% highlight bash %}
~/projects/ruby/blog  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
68f52b94aed6        blog_web            "bash -c 'bundle exec"   5 minutes ago       Up 3 minutes        0.0.0.0:3000->3000/tcp   blog_web_1
7dfa02f6c2de        redis:3.0.7         "docker-entrypoint.sh"   8 minutes ago       Up 3 minutes        0.0.0.0:6379->6379/tcp   blog_experiences_redis_1

{% endhighlight %}

After that, we need to enter in rails c, to do this we need to type 'docker exec -it <image_name> -it bash" and 'rails c'.

{% highlight bash %}
~/projects/ruby/blog  docker exec -it blob_web_1 bash
root@68f52b94aed6:/var/www/experiences# rails c
Running via Spring preloader in process 67
Loading development environment (Rails 5.0.0.1)

{% endhighlight %}

The gem Stoplight is very illustrative, it works like a stoplight (Mr. Obvious attacks again! kkk). Please look at the diagram below.

![stoplight]({{ site.url }}/assets/images/stoplightdiagram.png)

*Ps: I tried to extract the flux above from Stoplight code.*

Now, we will play with gem and after that we will try to use it inside our simple 'blog' code.

{% highlight ruby %}
root@e0e62e64b3ec:/var/www/experiences# rails c
Running via Spring preloader in process 49
Loading development environment (Rails 5.0.0.1)
irb(main):001:0> function = -> { puts "Hello World!" }
=> #<Proc:0x00560f121bc6b0@(irb):1 (lambda)>
irb(main):002:0> light = Stoplight('example') { function.call }
=> #<Stoplight::Light:0x00560f1219a498 @name="example", @code=#<Proc:0x00560f1219a4c0@(irb):2>, @cool_off_time=60.0, @data_store=#<Stoplight::DataStore::Memory:0x00560f118b6570 @failures=#<Concurrent::Map:0x00560f118b6548 entries=0 default_proc=#<Proc:0x00560f118b64f8@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/data_store/memory.rb:10>>, @states=#<Concurrent::Map:0x00560f118b6480 entries=0 default_proc=#<Proc:0x00560f118b6458@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/data_store/memory.rb:11>>, @lock=#<Monitor:0x00560f118b63e0 @mon_owner=nil, @mon_count=0, @mon_mutex=#<Thread::Mutex:0x00560f118b6390>>>, @error_handler=#<Proc:0x00560f118b6340@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:9 (lambda)>, @error_notifier=#<Proc:0x00560f118b6318@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:11 (lambda)>, @fallback=nil, @notifiers=[#<Stoplight::Notifier::IO:0x00560f118b62c8 @object=#<IO:<STDERR>>, @formatter=#<Proc:0x00560f118b62f0@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:15 (lambda)>>], @threshold=3>
irb(main):003:0> light.run
Hello World!
=> nil
irb(main):004:0> light.run
Hello World!
=> nil
irb(main):005:0> light.color
=> "green"
{% endhighlight %}

The code above shows how Stoplight works, to create a light we need to set an identifier, at this case 'example'. We also need to pass a code block that will be executed, in this case a lambda called 'function'. When we have a light, we can run it and the result of code block will be evaluated, we can get the light color too.

Ok, if code works everytime when called this gem is unuseful, but if you have a code that can fail you can use it to avoid some error scenarios. Let's take a look when something goes wrong.

{% highlight ruby %}
irb(main):010:0> function = -> { Blargh.new }
irb(main):011:0> light = Stoplight('error') { function.call }.with_fallback{|error| puts error; puts 'Fallback' }.with_cool_off_time(3)
=> #<Stoplight::Light:0x00560f11ed41a0 @name="error", @code=#<Proc:0x00560f11ed4240@(irb):11>, @cool_off_time=3, @data_store=#<Stoplight::DataStore::Memory:0x00560f118b6570 @failures=#<Concurrent::Map:0x00560f118b6548 entries=1 default_proc=#<Proc:0x00560f118b64f8@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/data_store/memory.rb:10>>, @states=#<Concurrent::Map:0x00560f118b6480 entries=0 default_proc=#<Proc:0x00560f118b6458@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/data_store/memory.rb:11>>, @lock=#<Monitor:0x00560f118b63e0 @mon_owner=nil, @mon_count=0, @mon_mutex=#<Thread::Mutex:0x00560f118b6390>>>, @error_handler=#<Proc:0x00560f118b6340@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:9 (lambda)>, @error_notifier=#<Proc:0x00560f118b6318@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:11 (lambda)>, @fallback=#<Proc:0x00560f11ed4178@(irb):11>, @notifiers=[#<Stoplight::Notifier::IO:0x00560f118b62c8 @object=#<IO:<STDERR>>, @formatter=#<Proc:0x00560f118b62f0@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:15 (lambda)>>], @threshold=3>
irb(main):012:0> light.run
uninitialized constant Blargh
Fallback
=> nil
irb(main):013:0> light.run

Fallback
=> nil
irb(main):014:0> light.run

Fallback
=> nil
irb(main):015:0> light.run
uninitialized constant Blargh
Fallback
=> nil
irb(main):016:0> light.run

Fallback
=> nil
irb(main):017:0> light.color
=> "yellow"
irb(main):018:0> light.run
uninitialized constant Blargh
Fallback
=> nil
irb(main):019:0> light.color
=> "red"
irb(main):020:0> sleep(3)
=> 3
irb(main):021:0> light.color
=> "yellow"
irb(main):022:0> light.run
uninitialized constant Blargh
Fallback
=> nil
irb(main):023:0> light.color
=> "red"
{% endhighlight %}

The code above is similar than previous, we added a function that always will fail (at least since we define Blargh class), we also added some customizations like 'with_fallback' and 'with_cool_of_time'. The function 'with_fallback' defines a custom fallback, this code will be evaluated when something goes wrong. The function 'with_cool_of_time' defines a timer to turn to yellow and the next call will define if light will be green or red again. Don't be afraid with this, Stoplight has a nice documentation and you can see many examples at project [page][stoplight].

When we have a huge application with multiple instances this code don't work well because Stoplight uses memory by default to store all light statuses. This gem already has a solution to it, we can use Redis to share lights statuses between the app instances. To do this we need to create a file at config/initializers and add the following code into it.

{% highlight bash %}
 ~/projects/ruby/blog touch config/initializers/stoplight.rb
{% endhighlight %}

{% highlight ruby %}
require 'stoplight'
require 'redis'

redis = Redis.new(host: ENV["REDIS"])

Stoplight::Light.default_data_store = Stoplight::DataStore::Redis.new(redis)
{% endhighlight %}

Now, we will use Stoplight in article#show route to responds with a fallback when something goes wrong. We need to add too the possibility to fail, to do it, we will add a code that will fail randomically. We need to create a command too. You can use all code directly on Article.find, but I will use a command to keep same pattern.

Create a file into lib/commands/article folder, this command will be responsible to search an article and to encapsulate a circuit breaker behavior. Add the following lines into created file.
{% highlight ruby %}
module Commands
  module Article
    class Find
      def call(id)
        ::Article.find(id)
      end
    end
  end
end
{% endhighlight %}

After that, we need to register a new command inside a Blog::Container.

{% highlight ruby %}
class Blog::Container
  extend Dry::Container::Mixin

  register('commands.article.create') do
    Commands::Article::Create.new
  end

  register('commands.article.find') do
    Commands::Article::Find.new
  end
end
{% endhighlight %}

And finally change the code responsible to find Article inside a ArticlesController to call our new Command.

{% highlight ruby %}
class ArticlesController < ApplicationController
  include AutoInject[create_article: 'commands.article.create',
                     find_article: 'commands.article.find'] #Adds a new dependency

  ...

  private
    def set_article
      @article = find_article.(params[:id]) # the . calls function #call inside a Commands::Article::Find
    end
  ...
end
{% endhighlight %}

Run 'docker-compose up' or 'rails s', create a article and see if everything works as expected.

Now we are ready to put a stoplight inside our project. First thing to do is add the fail possibility. We will use the following code to determine if it will fail or no.

{% highlight ruby %}
def fail?
  Random.rand < 0.5
end
{% endhighlight %}

After that, we need to add to code responsible responds by fallback. In our case, when something goes wrong fallback will responds with a fake Article.

{% highlight ruby %}
def fallback(id)
  ::Article.new(id: id, name: "Fallback", description: "Error happened Fallback response")
end
{% endhighlight %}

And add the code responsible to fail or search Article in database.

{% highlight ruby %}
def find(id)
  raise 'Something is wrong' if fail?
  ::Article.find(id)
end
{% endhighlight %}

When you search for a inexistent Article, the method #find will throw a error.

{% highlight ruby %}
irb(main):001:0> Article.find 239882
  Article Load (0.3ms)  SELECT  "articles".\* FROM "articles" WHERE "articles"."id" = ? LIMIT ?  [["id", 239882], ["LIMIT", 1]]
  ActiveRecord::RecordNotFound: Couldn't find Article with 'id'=239882
{% endhighlight %}

But you don't need to responds with a fallback in this case. To around this problem we need to add the code below.

{% highlight ruby %}
def custom_error_handler(error, handle)
  raise error if error.is_a?(ActiveRecord::RecordNotFound)
  handle.call(error)
end
{% endhighlight %}

And finally the code to get together all things.

{% highlight ruby %}
def call(id)
  Stoplight('article.find') do
    find(id)
  end.with_fallback do |error|
    fallback(id)
  end.with_error_handler do |error, handle|
    custom_error_handler(error, handle)
  end.run
end
{% endhighlight %}

All code will look like this:

{% highlight ruby %}
module Commands
  module Article
    class Find
      def call(id)
        Stoplight('article.find') do
          find(id)
        end.with_fallback do |error|
          fallback(id)
        end.with_error_handler do |error, handle|
          custom_error_handler(error, handle)
        end.run
      end

      private
      def fail?
        Random.rand < 0.5
      end

      def fallback(id)
        ::Article.new(id: id, name: "Fallback", description: "Error happened Fallback response")
      end

      def find(id)
        raise 'Something is wrong' if fail?
        ::Article.find(id)
      end

      def custom_error_handler(error, handle)
        raise error if error.is_a?(ActiveRecord::RecordNotFound)
        handle.call(error)
      end
    end
  end
end
{% endhighlight %}


Now, we can run our server again, and access a Article#show route. When you get a error the server will respond with a fallback, like the image below.

![stoplight]({{ site.url }}/assets/images/stoplightfallback.png)

If you check your log, you will see something like this:

![stoplight]({{ site.url }}/assets/images/stoplightlog.png)

We can add a custom notifier to something that you want when light changes the status from red to green or vice versa.

Bonus: We can add a custom notifier in our project, to do it, we need to create a new file and add the following content:
{% highlight ruby %}
touch lib/notifiers/custom.rb
{% endhighlight %}

{% highlight ruby %}
module Notifiers
  class Custom < Stoplight::Notifier::Base
    def notify(light, from_color, to_color, error)
      puts("[Custom] - Light: #{light.inspect} - From color: #{from_color} - To color #{to_color} - #{error}")
    end
  end
end
{% endhighlight %}

We only need to keep the same method signature of notify and add the custom notifier in stoplight configuration. To configure stoplight, we need to add the line below inside config/initializers/stoplight.rb.

{% highlight ruby %}
Stoplight::Light.default_notifiers += Array(Notifiers::Custom.new)
{% endhighlight %}

When we call the route again until an error occurs. The log will look like this:

{% highlight ruby %}
web_1                | Switching article.find from green to red because RuntimeError Something is going wrong
web_1                | [Custom] - Light: #<Stoplight::Light:0x0056395bbf8368 @name="article.find", @code=#<Proc:0x0056395bbf84f8@/var/www/experiences/lib/commands/article/find.rb:5>, @cool_off_time=60.0, @data_store=#<Stoplight::DataStore::Redis:0x0056395be41de0 @redis=#<Redis client v3.3.2 for redis://redis_experiences:6379/0>>, @error_handler=#<Proc:0x0056395bbf8278@/var/www/experiences/lib/commands/article/find.rb:9>, @error_notifier=#<Proc:0x0056395b7d8e18@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:11 (lambda)>, @fallback=#<Proc:0x0056395bbf82f0@/var/www/experiences/lib/commands/article/find.rb:7>, @notifiers=[#<Stoplight::Notifier::IO:0x0056395b7d8dc8 @object=#<IO:<STDERR>>, @formatter=#<Proc:0x0056395b7d8df0@/usr/local/bundle/gems/stoplight-2.1.0/lib/stoplight/default.rb:15 (lambda)>>, #<Notifiers::Custom:0x0056395be297e0>], @threshold=3> - From color: green - To color red - Something is going wrong
{% endhighlight %}

Stoplight has a panel to displays all light statuses and to do some actions like lock light status. If you are interested in it, please take a look at the project [page][stoplight-admin].

You can download all code showed from [here][experiences-final]. That's all. If you have any doubt please post it in commentaries section.

Thanks!

#### References
* https://github.com/Netflix/Hystrix
* http://electronics.howstuffworks.com/circuit-breaker2.htm
* https://github.com/orgsync/stoplight
* https://github.com/jnunemaker/resilient
* https://github.com/yammer/circuitbox
* https://www.infoq.com/br/presentations/exercising-failure-at-netflix#downloadPdf

[homecircuitbreaker]: http://electronics.howstuffworks.com/circuit-breaker2.htm
[stoplight-admin]: https://github.com/orgsync/stoplight-admin
[stoplight]: https://github.com/orgsync/stoplight
[resilient]: https://github.com/jnunemaker/resilient
[circuitbox]: https://github.com/yammer/circuitbox
[experiences]: https://github.com/GabrielMalakias/experiences/releases/tag/0.0.2
[experiences-final]: https://github.com/GabrielMalakias/experiences/releases/tag/0.0.3
[kolton-andrus]: https://www.infoq.com/br/presentations/exercising-failure-at-netflix#downloadPdf
