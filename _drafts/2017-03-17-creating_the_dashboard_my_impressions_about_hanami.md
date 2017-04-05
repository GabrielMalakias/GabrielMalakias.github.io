---
layout: post
title: "IoT Saga - Part 2 - Creating the dashboard: My impressions about hanami"
date:   2017-03-17 12:00:00
categories: ruby hanami iot
disqus: true
description: I intend to show some good points to choose Hanami as Framework
---

Hello everyone, Today I'm gonna talk what I've been learning and so on. My first language at college was Java, all professors made me learn many things about Software Engineering and Design Patterns and also different than a many developers that I met, I enjoyed so much my course.

When I decided to learn Ruby, I started with Rails the most common choice, Rails is great, this incredible framework allows to you build things very fast but some applications that I worked on could be better with using a different mindset, don't get me wrong I really like Rails but we can try another framework, language or paradigm. I decided to experiment Hanami because is different and enforces me to learn new tools and approaches.


### About the project

I intend to build a kind of dashboard where I can manage and send commands my arduino sensors and actuators. To reach my objective I'm going to create three applications (Usgard, Mygard and Bitfrost).

The first one will be the dashboard, I renamed the project to Usgard (the first saga post mentions SpaceWing as project name) because it's a name that makes me remember Asgard where Nordic Gods live, from there Gods can send commands and collect information from another kingdown like Midgard. The second one, called Mygard, can receive commands and send status to somewhere. The third one will be the Bitfrost, this application will be responsible to connect Mygard and Usgard.

There is a diagram to explain my idea:

![usgard_perspective]({{ site.url }}/assets/images/usgard_perspective.png)

Well, as you've seen above I intend combine different technologies to reach connect all application, I want to combine this Stack because I've already saw how can I combine them but I intend to change all pieces to learn and discover new ways to build applications.

Usgard uses Hanami as WebFramework and PahoJS as WebSocket connector. The JS structure I didn't find a good framework for my purposes yet but I would use React, Redux or something that I don't know. :)

I created two entities to manage my data, I called the first one Sensor and the another one Actuator, In the show page we can see the Sensor/Actuator information and send some message in actuator's page. I didn't finish my first version yet but I believe that I learnt some new things using Hanami and I will try to share it with you.

### My opinion about Hanami

###### 1. Decreases the coupling between domain and persistence layer

Well, I usually saw examples like the following diagram at college:

![dao-java]({{ site.url }}/assets/images/java_dao_example.gif)
http://www.corej2eepatterns.com/DataAccessObject.htm

Now I can remember how is boring to write all DAO(Data Acess Object) implementations but I was just starting, by the time I didn't know any technology capable to it by itself. After that I learnt just a little of SpringJPA and also when I was learning it I saw something like the following code:

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

```shell
~/projects/ruby/usgard(dev ✔) bundle exec hanami generate model blargh

      create  lib/usgard/entities/blargh.rb
      create  lib/usgard/repositories/blargh_repository.rb
      create  db/migrations/20170402190609_create_blarghs.rb
      create  spec/usgard/entities/blargh_spec.rb
      create  spec/usgard/repositories/blargh_repository_spec.rb
```

From my point of view the DAO use is really good because we can remove the responsability to manage our database from models. This mindset can keep our models more Thin and can guide us to Single Responsability principle.

##### 2. Views

Using Rails is common use Helpers to create anything you want to do inside template however this custom methods are shared inside the application.

"The Rails framework provides a large number of helpers for working with assets, dates, forms, numbers and model objects, to name a few. These helpers are available to *all templates* by default." - Rails doc

Well, using Hanami we have Views Layer the Hanami definition for Views is that:




"A view is an object that encapsulates the presentation logic of a page. A template is a file that defines the semantic and visual elements of a page. In order to show a result to a user, a template must be rendered by a view."

##### 3.Env files

By default Hanami favors Env files use.

##### 4. Controllers

Hanami uses Actions division instead Controllers with many actions. Inside this Layer the framework allows you to use validations, Exceptions Management and callbacks.

##### 5. Umbrella applications

Sometimes you need to create application sharing the same common code (at least the same persistence and domain layer), Hanami allows you to create applications inside the main application using the Umbrella concept. This concepts allows we to create an api, admin and web applications at same codebase.


