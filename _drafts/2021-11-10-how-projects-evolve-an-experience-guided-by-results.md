---
layout: post
title: 'How projects evolve - An experience guided by results'
date:   2021-11-11 00:00:00
categories: ruby sidekiq elixir actormodel
disqus: true
description: Sidekiq
cover: /assets/images/how_projects_evolve.jpg
---

Systems, languages and projects come and go as people progress in their own careers, solutions that were not so obvious in the beginning might become easier to tackle as project and team evolves. In this post I would like to share a bit about my experience on scaling a system that I worked with by observing and trying to understand the problem before thinking about changing the technology completely or even choosing the wrong one again.

While working at Delivery Hero I had the opportunity to migrate a system to a new one at that point it was decided to keep the same language but trying to avoid bottlenecks. One of multiple problems that prevented initial solution from scaling nicely was a loop that checks every single delivery and rider applying or not actions based on configurations the app allows.

Unfortunately I cannot share the exact same code but I created something that represents the problem pretty well however I will try to guide you through the process with an example talking about the past, present and what was my idea about the future.

#### The past

Imagine you are building a software that keeps track of a mobile device's position. One of the reasons why the solution has to check is that we would like to identify as soon as possible if the device is off applying a series of rules. To handle that, our initial solution loops through a list of devices checking if the latest location was received recently, so the app can change the device status to `out_of_range` and in case its out of range for two periods in a row it marks the device as `offline`.

{% highlight ruby %}
  def call
    filter_online.each do |device|
      if device.no_recent_location_received?
        if device.is?('out_of_range')
          device.move_to('offline')
        else
          device.move_to('out_of_range')
        end
      else
        device.move_to('online')
      end
    end
  end
{% endhighlight %}

So by simply executing the method above every 1 minute we could easily solve the problem. Nice, but wait what are the problems within this simple solution.

Based in my experience I can enumerate a few:

1. If a single device returns an error or the app crashes the devices which were not checked yet will have to the next period to be processed;
2. Given that the solution sequential its impossible to distribuite the load across different instances, pods or threads;
3. If the time to check each device increases it might reach a point where we cannot check all devices within a minute making it cascade to the next period;

I might have lost more problems but I think the idea its pretty clear here, it doesnt scale, because we cannot divide and conquer and its quite fragile. How can we make it better?

### The present

One of the things I've been experimenting with its elixir, it has a great actor model system however sometimes I feel there is a certain resistance on adopting it. Well the version 2 of the problem mentioned before was written in Ruby using Rails also as framework, I really love ruby and really believe that its quite mature offering stable and awesome solutions for 90% of companies. So how was it implemented?

What if each device could be checked individually with its own "process", not affecting any others without changing the language and using only sidekiq? That would be pretty cool right?

The proposed solution would like a recursive call using Sidekiq to treat errors using it's retry mechanism and awesome `reliable_scheduler`(Sidekiq Pro)

{% highlight ruby %}
RunCheck.perform_in(1.minute, id)

def call(id)
  device = find(id)
  if device.no_recent_location_received?
    if device.is?('out_of_range')
      device.move_to('offline')
    else
      device.move_to('out_of_range')
    end
  else
    device.move_to('online')
  end
end
{% endhighlight %}

So lets now go back to the problems we had in the initial solution to validate if new solution is better or worse.

About the first problem, since now each device is managed by its own job individually it can no longer affect any other or even stop the process, so thats a win. The solution above can easily scale using Kubernetes HPA to increase the number of pods as the number of jobs increases by checking cpu levels, so it scales a bit better. Last point, the time to check each device might increase as we add more checks but its pretty unlikely to cross the minute barrier. "Wunderbar"(Marvelous) as germans say, but is it the best solution ever?

No, its not, as you might know Postgres has a limit in number of connections and as we add more and more pods to handle the increasing number of pods at some point it will be just unbearable to the DB.

Oh my (MEME)

Wait, there is a way to solve that, the solution we found was using PGBouncer in transaction mode, so each query to the DB executes quite fast creating like a ConnectionPool for all the pods.

Cool all problems solved right? Temporarely, this solution scales until a certain point due to DB constraits, at some point the DB will not me able to handle these many parallel requests and will just slow everything down.

Damm it!

### The desired future

The problem faced now its not about the language itself, but instead the approach itself, remember the actor model? Well if the app didnt have to search things in the DB for every single job and process the problem would be avoided. How would that look like?

Instead of relying on the DB 100% of the time the state could be kept in memory using someting like genservers in elixir or even redis, however I believe it would be better levereged by in memory actor model system.

However the solution using genservers is not perfect because it eliminates the round-trips to the DB but it might increase the memory consumption considerably. Another point is that the state would have to be pretty well managed to avoid unexpected bugs or problems but I still believe that's a best solution compared to the Sidekiq loops.


### Conclusion

Rewriting an app is something costly and it also demands time, not only due to the rewriting itself but also for the project to reach the maturity. It should also never be only to use the fanciest brand new languaged that has just appeared backed by Google, furthermore I strongly believe that programming languages are tools in our utility belt and even though we as developers we have our preferences (me included, I :heart Ruby) we should always be accontable to use the best tool to the right job.

Cover Photo by <a href="https://unsplash.com/@chrislawton?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Chris Lawton</a> on <a href="https://unsplash.com/s/photos/seasons?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
