---
layout: post
categories: ruby circuit-breaker
---


At this year I went to QCon São Paulo, I saw many cool things and lots of new technologies. At the conference some speakers talked about how we can make resilient systems. Kolton Andrus spoke a lot about of Netflix Toolset, he mentioned Eureka, Ribbon, Archaius and Hystrix. I liked so much of Hystrix, because the concept behind this mechanism sounds very simple and can be useful in many situations. In this post I will try to talk about it.

First, what's is Hystrix? The Netflix describes Hystrix as "Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.". Behind all advantages that it can give to us, has a lot of patterns but we will talk about only one, called of Circuit-Breaker.

First, we need to think about circuit-breaker. Let me see... Oh Circuit-Breaker pattern works like a home circuit-breaker, Obvious. But how it works? You can see it [here][homecircuitbreaker]. In IT world, this mechanism can be applied to some cases to prevent errors or latency. Picture this, you has a call to a external service many things can fail, network, server down or something unexpected. If you wanna to make your system more resilient you need to assume that your system can fail and after that add some, and responds a default or a cache response if a service is down. Using an circuit-breaker when something goes wrong N times, the circuit-breaker(opened) stops to call the external service(or a failing method) and pass to responds only with a fallback method. After a time period, the circuit-breaker close and try to call a external service again, it can works or not.

In this post I will try to show some examples with the same app at the previous post, you can download [here][experiences]. I added Docker in this project only to turn easier the environment setup, you can use only ruby and rails to run.

I will show only one gem available in Ruby, but if you search at RubyToolbox site you can found many similar solutions like [circuitbox][circuitbox] or [resilient][resilient].In this post we will use the [stoplight][Stoplight] gem.

Ok how it works? Well, we will do it together along this post it's easier to understand with examples (at least for me :) ).

*Ps: To run with docker you can use 'docker-compose up' and after that 'docker exec -it <image-name/id> bash' to enter in bash. After that it only type 'rails c'.*

First thing that we need to do is add gem in project, to do this we will add this line in Gemfile.

{% highlight ruby %}
gem 'stoplight'
{% endhighlight %}
*Ps: Everytime that we add a new gem in Gemfile, we need to run 'docker-compose build' again to after that run 'docker-compose up'.*

In the project folder we need to run Docker and type the commands below to enter in rails console.

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

The command above build the image from Dockerfile, starts all dependencies and executes a command specified in docker-compose (in our case 'bundle exec puma').


{% highlight bash %}
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

After that, we need to enter in rails c, to do this we typed 'docker exec -it <image_name> -it bash" and 'rails c'.

The gem Stoplight is very illustrative, it works like a stoplight (Mr. Obvious attacks again! kkk). Please look at the diagram below.

![stoplight]({{ site.url }}/assets/images/stoplightdiagram.png)

At the example above I tried to extract from Stoplight code. It works basically like a diagram above.

Now, we will play with gem and after that we will try to use in our simple 'blog' code.

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

The code above shows how Stoplight works, to create a light we need to put an identifier, at this case 'example', and the code that will be executed (function variable). When we have a light we can run it anf get color. Ok, if code works everytime when called this gem is unuseful, but if you have a code that can fail you can use this. Take a look how gem behaves with a failing code.

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

The code above is similar than previous code, we added a function that will always fail (at least since we define Blargh class), we also added some options like 'with_fallback' and 'with_cool_of_time'. The 'with_fallback' defines a custom fallback when something is going wrong and 'with_cool_of_time' defines a timer to turn to yellow, don't be afraid with this, Stoplight has a nice documentation and you can see many examples at project [page][stoplight].

When we have a huge application with multiple instances this code don't work well because Stoplight uses memory by default, but this gem already has a solution to this. We can use Redis to share lights statuses between instances. To do this we need to create a file at config/initializers add the following code into stoplight.rb


{% highlight bash %}
 ~/projects/ruby/blog touch config/initializers/stoplight.rb
{% endhighlight %}

{% highlight ruby %}
require 'stoplight'
require 'redis'

redis = Redis.new(host: ENV["REDIS"])

Stoplight::Light.default_data_store = Stoplight::DataStore::Redis.new(redis)
{% endhighlight %}

Now, we will use Stoplight in article#show route to responds with a fallback when something is going wrong. We need to add too a possibility to fail, to do this we will add a code that will fail randomically.

First we need to create a command, similar to a previous post. You can use all code directly on Article.find, but I will use a command to keep the previous pattern.

Create a file into lib/commands/article folder this command will be responsible to search and encapsulates a circuit breaker behavior. Add the following lines into the file.
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

Now we are prepared to put a stoplight inside our project. First thing to do is add the fail possibility. We will use the following code to determine if it will fail or no.

{% highlight ruby %}
def fail?
  Random.rand < 0.5
end
{% endhighlight %}

After that we need to add to code responsible responds for fallback. In our case, when something goes wrong fallback will responds with a fake Article.

{% highlight ruby %}
def fallback(id)
  ::Article.new(id: id, name: "Fallback", description: "Error happened Fallback response")
end
{% endhighlight %}

And add the code responsible to fail or search Article in database.

{% highlight ruby %}
def find(id)
  raise 'Something is going wrong' if fail?
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
        raise 'Something is going wrong' if fail?
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

That's all. If you have any doubt please post it in commentaries section.

Thanks!

[homecircuitbreaker]: http://electronics.howstuffworks.com/circuit-breaker2.htm
[stoplight]: https://github.com/orgsync/stoplight
[resilient]: https://github.com/jnunemaker/resilient
[circuitbox]: https://github.com/yammer/circuitbox
[experiences]: https://github.com/GabrielMalakias/experiences
