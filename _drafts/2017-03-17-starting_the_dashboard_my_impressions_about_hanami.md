---
layout: post
title: "IoT Saga - Part 2 - Starting the dashboard: My first impressions about hanami"
date:   2017-03-17 12:00:00
categories: ruby dry-rb rails
disqus: true
description: I intend to show some good points to choose Hanami as Framework
---

Hello everyone, Today I'm gonna talk about Hanami from my point of view a awesome Framework that allows to you learn a bunch of new things. My first language at university was Java, all professors made me learn a lot of Software Engineering and Design Patterns, I enjoyed so much it. When I decided to learn Ruby, I saw the amazing powers of ActiveRecord and the Metaprogramming powers I was so excited but with great powers comes great responsabilities. I think Rails is great, this incredible framework allows to you build things very fast but the some applications that I worked could be better with using a different mindset.

Well, I usually saw examples like the following diagram at university:

![dao-java]({{ site.url }}/assets/images/java_dao_example.gif)
http://www.corej2eepatterns.com/DataAccessObject.htm

I remember how is boring to write all DAO implementations because I was just starting, by the time I didn't know any tecnology capable to it by itself. After that I learnt just a little of SpringJPA and also when I was learning it I saw this kind of code:

```java
public interface UserRepository extends CrudRepository<User, Long> {
  Long countByLastname(String lastname);
}
```

This example made me think, why do they use Repository to search instead use an ORM like us (Ruby on Rails programmers)?. I think we use ActiveRecord because is natural for us and it can solve our problems however sometimes it causes some problems.




