---
layout: post
title: "IoT Saga - Part 2 - Creating the dashboard: My little overview about hanami"
date:   2017-03-17 12:00:00
categories: ruby hanami iot
disqus: true
description: I intend to show some good points to choose Hanami as Framework
---

Hello everyone, Today I'm gonna talk what I've been learning and so on. My first language at college was Java, all professors made me learn many things about Software Engineering and Design Patterns and also different than a many developers that I met, I enjoyed so much my course.

When I decided to learn Ruby, I started with Rails the most common choice, Rails is great, this incredible framework allows to you build things very fast, even through some applications that I worked on could be better with using a different mindset, don't get me wrong I really like Rails but we can try another framework, language or paradigm. I decided to experiment Hanami because is different and enforces me to learn new tools and approaches.


### About the project

I intend to build a kind of dashboard where I can manage and send commands my arduino sensors and actuators. To reach my objective I'm going to create three applications (Usgard, Mygard and Bitfrost).

The first one will be the dashboard, I renamed the project to Usgard (the first saga post mentions SpaceWing as project name) because it's a name that makes me remember Asgard where Nordic Gods live, from there Gods can send commands and collect information from another world like Midgard. The second one, called Mygard, can receive commands and send status to somewhere. The third one will be the Bitfrost, this application will be responsible to connect Mygard and Usgard.

There is a diagram to explain my basic idea (I intend to change the architeture):

![usgard_perspective]({{ site.url }}/assets/images/usgard_perspective.png)

Well, as you've seen above I intend combine different technologies to connect all applications, I want to combine this Stack because I've already saw how can I combine them but I intend to change all pieces to learn and discover new ways to build applications.

Usgard uses Hanami as WebFramework and PahoJS as WebSocket connector. The JS structure I didn't find a good framework for my purposes yet, however I would use React, Redux or something that I don't know. :)

I created two entities to manage my data, I called the first one Sensor and the another one Actuator, In the show page we can see the Sensor/Actuator information and send some message in actuator's page. I didn't finish my first version yet, but I believe that I learnt some new things using Hanami and I will try to share it with you.

### The bright side

##### DISCLAIMER
***First of all, this post represents only my opinion about the topics discussed, I do not intent to hurt anyone or something else. Think by yourself and get your own conclusions. I really like to learn new technologies and I feel it can push me forward.***

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
This example made me think, why do they use Repository to search instead use an ORM like us (Ruby on Rails programmers)?. I think we use ActiveRecord because is natural for us and it can solve our problems however can create different ones.

Using Hanami models we have:

***"Entity - A model domain object defined by its identity.
Repository - An object that mediates between the entities and the persistence layer."*** - Hanami Github

We also can create models and repositories using the following command:

```shell
~/projects/ruby/usgard(dev ✔) bundle exec hanami generate model blargh

      create  lib/usgard/entities/blargh.rb
      create  lib/usgard/repositories/blargh_repository.rb
      create  db/migrations/20170402190609_create_blarghs.rb
      create  spec/usgard/entities/blargh_spec.rb
      create  spec/usgard/repositories/blargh_repository_spec.rb
```

From my point of view, the concept DAO is really good because we can remove the responsability to manage our database from models. This mindset can keep our models simpler and guide us to Single Responsability principle.

##### 2. Views

***"The Rails framework provides a large number of helpers for working with assets, dates, forms, numbers and model objects, to name a few. These helpers are available to ALL TEMPLATES by default.***

***In addition to using the standard template helpers provided, creating custom helpers to extract complicated logic or reusable functionality is strongly encouraged."*** - Rails doc

"A Hanami view is an object that defines presentational logic. Helpers are modules designed to enrich views with a collection of useful features." - Hanami doc

I usually saw Rails applications using Helpers to create anything you want to do into template like forms or custom structures, however these custom methods are shared for all application then I recomment caution. I know Rails Helpers is different to the concept of Hanami Views even through concept of View layer. Well, the Hanami definition for Views is that:

"A view is an object that encapsulates the presentation logic of a page. A template is a file that defines the semantic and visual elements of a page. In order to show a result to a user, a template must be rendered by a view."

Using views you can build all pages using the Hanami DSL or mix some content built using Html with the code generated by DSL.

##### 3.Env files

"Another approach to config is the use of config files which are not checked into revision control, such as config/database.yml in Rails. This is a huge improvement over using constants which are checked into the code repo, but still has weaknesses: it’s easy to mistakenly check in a config file to the repo; there is a tendency for config files to be scattered about in different places and different formats, making it hard to see and manage all the config in one place. Further, these formats tend to be language- or framework-specific.

The twelve-factor app stores config in environment variables (often shortened to env vars or env). Env vars are easy to change between deploys without changing any code; unlike config files, there is little chance of them being checked into the code repo accidentally; and unlike custom config files, or other config mechanisms such as Java System Properties, they are a language- and OS-agnostic standard." - 12 factor

In last post I show my application setup using Docker, from my point of view, Hanami enforces you to use EnvVars instead config files, it's good by many reasons as you've seen above, but it turned my setup easier and if you think about Environment automatization and Virtualization this concept would be good.

##### 4. Controllers

I strongly believe is it the best point of Hanami. Hanami uses Actions instead Controllers with many actions. The Action Layer allows us to use validations, exceptions management, exposures(to grant isolation) and callbacks.


``` ruby
module Web::Controllers::Sensors
  class Create
    include Web::Action # Action mixin

    # Dry-AutoInject \o/, I used but it isn't default
    include ::AutoInject['commands.sensor.create']


    # Input validation using the power of Dry-Validation
    params do
      required(:sensor).schema do
        required(:name).filled(:str?)
        required(:mqtt_topic).filled(:str?)
      end
    end

    # Custom status returned when an Exception is raised
    handle_exception ArgumentError => 422


    # Single method to execute the action
    def call(params)
      if params.valid?

        # Similar to Interactions, strongly recommended by Hanami Team
        sensor = create.(params.get(:sensor))

        redirect_to routes.sensor_url(id: sensor.id)
      else
        raise ::ArgumentError, params.errors
      end
    end
  end
end
```

This bunch of powerful tools allows us to create customized abstractions before delegates to another layer, in my case Commands(Interaction) Layer.

##### 5. Umbrella applications

Sometimes you need to create application sharing the same common code (at least the same persistence and domain layer), Hanami allows you to create applications inside the main application using the Umbrella concept. This concepts allows we to create an api, admin and web applications at same codebase.

***We know that the set of features that we're going to introduce doesn't belong to our main UI (Web). On the other hand, it's too early for us to implement a microservices architecture, only for the purpose of helping our users reset their password.*** - Hanami Guide

The Premature Optimization problem applies here. Usually some startup or a company project wants to be released as fast as we can, on the other hand you really prefer to build isolated application, even through you do not have time enough to create microservices, if it applies to you Hanami would be a really good start.


### The not so bright side

Well, when I was building my first test application (Usgard is my second one), I faced two problems. The first one was *How can I run sidekiq?*. Was suffice a quick look at the project issues and I saw my answer: https://github.com/hanami/hanami/issues/695.

The second one was about how hanami mount routes.

```ruby
mount Sidekiq::Web,     at: '/sidekiq'
mount Api::Application, at: '/api'
mount Web::Application, at: '/'
```

Maybe when was your first contact you can commit a mistake and change the root route order as the following example.

```ruby
mount Web::Application, at: '/'
mount Sidekiq::Web,     at: '/sidekiq'
mount Api::Application, at: '/api'
```

If you do it mistakenly, well perhaps your '/sidekiq' route can be inacessible. I understand it, this 'error' occurs because of the manner how Hanami build the application routes, but I wasted at least one hour to discover it.

Using Hanami maybe you can feel alone or maybe cannot find your answer on internet, in these cases you can go to Hanami Chat to request for help or take a look at the codebase.

I think Rails is great and Hanami too. Hanami will release the 1.0.0 pretty soon and a new core team was created then I really believe in this project.

Thanks @Hanami and @Rails teams and contribuitors!

### References

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/
https://martinfowler.com/eaaCatalog/repository.html
http://hanamirb.org/guides/views/overview/
http://hanamirb.org/guides/models/overview/
http://hanamirb.org/guides/actions/overview/
https://12factor.net/config
http://api.rubyonrails.org/classes/ActionController/Helpers.html
http://hanamirb.org/guides/architecture/overview/
https://martinfowler.com/bliki/MonolithFirst.html
https://github.com/hanami/hanami/issues/695.
https://gitter.im/hanami/chat
https://mkdev.me/en/posts/a-couple-of-words-about-interactors-in-rails

