---
layout: post
title: "IoT Saga - Part 2 - Starting the dashboard: My impressions about hanami"
date:   2017-03-17 12:00:00
categories: ruby dry-rb rails
disqus: true
description: I intend to show some good points to choose Hanami as Framework
---

Hello everyone, Today I'm gonna talk what I've been learning and so on. My first language at college was Java, all professors made me learn many things about Software Engineering and Design Patterns, different than a many developers I saw, I enjoyed so much my course.

When I decided to learn Ruby, I started with Rails the most common choice, Rails is great, this incredible framework allows to you build things very fast but the some applications that I worked could be better with using a different mindset I decided to make an application using Hanami.

### Introduction - Some good points about Hanami

###### 1. Decrease coupling between domain and persistence layer

Well, I usually saw examples like the following diagram at college:

![dao-java]({{ site.url }}/assets/images/java_dao_example.gif)
http://www.corej2eepatterns.com/DataAccessObject.htm

Now I can remember how is boring to write all DAO implementations but I was just starting, by the time I didn't knew any technology capable to it by itself. After that I learnt just a little of SpringJPA and also when I was learning it I saw this kind of code:

```java
public interface UserRepository extends CrudRepository<User, Long> {
  Long countByLastname(String lastname);
}
```
This example made me think, why do they use Repository to search instead use an ORM like us (Ruby on Rails programmers)?. I think we use ActiveRecord because is natural for us and it can solve our problems however sometimes it causes some problems.

Using Hanami-models we have:

"Entity - A model domain object defined by its identity.
Repository - An object that mediates between the entities and the persistence layer." - Hanami Github

We can create models and repositories using the following command:

~/projects/ruby/usgard(dev ✔) bundle exec hanami generate model blargh

      create  lib/usgard/entities/blargh.rb
      create  lib/usgard/repositories/blargh_repository.rb
      create  db/migrations/20170402190609_create_blarghs.rb
      create  spec/usgard/entities/blargh_spec.rb
      create  spec/usgard/repositories/blargh_repository_spec.rb

##### 2. Views

Using Rails is common use Helpers to create anything you want to do inside template and all methods are shared in all application. Well, using Hanami we have Views Layer the Hanami definition for Views is that:

"A view is an object that encapsulates the presentation logic of a page. A template is a file that defines the semantic and visual elements of a page. In order to show a result to a user, a template must be rendered by a view."

##### 3.Env files

By default Hanami favors Env files use.

##### 4. Controllers

Hanami uses Actions division instead Controllers with many actions. Inside this Layer the framework allows you to use validations, Exceptions Management and callbacks.

##### 5. Umbrella applications

Sometimes you need to create application sharing the same common code (at least the same persistence and domain layer), Hanami allows you to create applications inside the main application using the Umbrella concept. This concepts allows we to create an api, admin and web applications at same codebase.


