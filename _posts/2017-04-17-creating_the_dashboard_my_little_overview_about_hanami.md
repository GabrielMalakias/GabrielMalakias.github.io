---
layout: post
title: "IoT Saga - Part 2 - Creating the dashboard: My little overview about hanami"
date:   2017-04-17 00:00:00
categories: ruby hanami iot
disqus: true
description: I intend to show some good points to choose Hanami as Framework
---

Hello, Today I will talk about I’ve been learning. My first language at college was Java, from my point of view this language enforces you to learn many things.  By the time, I remember how many hours I spent learning Software Engineering and Design Patterns to build all requested homework. Even through different than a many developers that I met, I enjoyed so much my course.

When I decided to learn Ruby I also started with Rails (I'm still using it everyday) the most common choice. Rails is great, this incredible framework allows us to build applications very fast. It uses Convention over Configuration and combine a very powerful toolkit to improve our productivity. On the other hand, some applications that I worked on could be better with using a different mindset. Don’t get me wrong, I like Rails but we can try another framework, language or paradigm.

I decided to experiment Hanami, because I saw a good talk of [Marcello Rocha][marcello-rocha] on RubyConf 2016 and also I can learn new approaches. I will share my throughts about Hanami along this post.

### About the project

I intend to build a kind of dashboard where I can manage and send commands my arduino sensors and actuators. To reach my goal I'm going to create three applications (Usgard, Mygard and Bitfrost).

The first one will be the dashboard, [Usgard][usgard]. The first part of this saga mentions [SpaceWing][saga-part-one] as project name but I decided to rename it. I chose this name because makes me remember Asgard, where Nordic Gods live. From there Gods can send commands and collect informations from worlds like Midgard. The second one, called Mygard, will receive commands and send data to RXTX. The third one will be the Bitfrost, the application to connect Mygard and Usgard.

There is a diagram to explain my basic idea (I intend to change the architeture):

![usgard_perspective]({{ site.url }}/assets/images/usgard_perspective.png)

Well, as you’ve seen above, I intend combine different technologies to create this dashboard. I want to combine this Stack because Java sounds familiar to me and a currently I know how I can combine them. My main aim is build a sandbox where I can test and create new things.

Usgard uses Hanami as WebFramework and PahoJS as WebSocket connector. I didn't find something cool to my JS yet, I would use React, Redux or something that I don't know. :) Suggestions?

I created two entities to manage my data, I called the first one Sensor and the another one Actuator. On the show page we can see the Sensor/Actuator information and send messages in actuator’s page. I didn’t finish the first version yet, but I learnt some new things using Hanami and I will share it with you.

### The bright side

##### DISCLAIMER
***First of all, this post represents only my opinion about the topics discussed, I do not intent to hurt anyone or something else. Think by yourself and get your own conclusions. I like to learn new ways to build applications and it can push me forward.***

###### 1. Decreases the coupling between domain and persistence layer

Well, I usually saw examples like the following diagram at college:

![dao-java]({{ site.url }}/assets/images/repository.gif)
* *Extracted from: https://martinfowler.com/eaaCatalog/repository.html*

Now, I can remember how was boring to write all Repository implementations using pure JDBC only to populate my POJO classes, but I was starting. By the time I didn’t know any technology capable to it by itself. After that I learnt a little of SpringJPA then when I was learning it, I saw something like the following code:

```java
public interface UserRepository extends CrudRepository<User, Long> {
  Long countByLastname(String lastname);
}
```
This example made me think. Why do they use Repository to search instead use an ORM like us (Ruby on Rails programmers)? Although we use ActiveRecord because is natural for us and usually solve our problems very well, we can try something different.

Using Hanami models we have:

***"Entity - A model domain object defined by it's identity.
Repository - An object that mediates between the entities and the persistence layer."*** - [Hanami documentation][hanami-models-doc]

We can create models and repositories using a command or create it by ourselves. Running the command below, Hanami creates the entity, the repository to manipulate data and the migration file. Inside it, we can describe the table columns and types to represent our Entity.

```shell
~/projects/ruby/usgard(dev ✔) bundle exec hanami generate model blargh

      create  lib/usgard/entities/blargh.rb
      create  lib/usgard/repositories/blargh_repository.rb
      create  db/migrations/20170402190609_create_blarghs.rb
      create  spec/usgard/entities/blargh_spec.rb
      create  spec/usgard/repositories/blargh_repository_spec.rb
```

From my point of view, the Repository pattern is good because removes the responsibility to manage database from models. Moreover this pattern can keep our models simpler and guide us to the [Single Responsibility principle][toptal-s].

##### 2. Views

***"The Rails framework provides a large number of helpers for working with assets, dates, forms, numbers and model objects, to name a few. These helpers are available to ALL TEMPLATES by default.***

***In addition to using the standard template helpers provided, creating custom helpers to extract complicated logic or reusable functionality is strongly encouraged."*** - [Rails documentation][rails-helpers-doc]

I usually saw Rails applications using Helpers to create anything sharing these custom methods along the application. I know Rails Helpers is different to the concept of Hanami Views but view layer is a good addiction to encapsulates only the necessary code. Well, the Hanami definition for Views is that:

***"A view is an object that encapsulates the presentation logic of a page. A template is a file that defines the semantic and visual elements of a page. In order to show a result to a user, a template must be rendered by a view."*** - [Hanami documentation][hanami-views-doc-description]

##### 3.Env files

According to [Twelve-Factor][twelve-factor-config], config files as database.yml in Rails, is a big improvement instead constants along application. On the other hand, it’s easy to send these files mistakenly to code repo and also every language adopt a different format each other.
The Twelve-Factor suggest to us, to use Environment variables instead config/*.yml to decrease the chance to send to credentials to the code repo. Furthermore, it’s a “language- and OS-agnostic standard”

At [part one][saga-part-one], I showed how I made my application setup using Docker. Using env-vars instead config files turned it easier. Besides that, if you think about Environment automatization and Virtualization this concept would be good.

##### 4. Controllers

I strongly believe it's the best point of Hanami. Hanami uses Actions instead Controllers. The Action Layer allows us to:

* ***Create whitelisting:*** to permit only trusted data;
* ***Create custom validations:*** Hanami::Validation uses the power of Dry-Validation as dependency then we can build our validations
* ***Create custom exception management:*** We can change the status or execute an method, for example;
* ***Expose our variables using exposures:*** to grant encapsulation;
* ***Use callbacks:*** I tend to avoid it, but some times we need to execute a method before Action#call;

And many other things, you can check it at [project page][hanami-controllers].

I created an action in my project and I used the following structure to all actions, at least now.

``` ruby
module Web::Controllers::Sensors
  class Create
    include Web::Action # Action mixin (Composition over Inheritance)

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

        # Similar to Interactions, they use something similar at Dnsimple, you can check at: https://blog.dnsimple.com/2017/03/why-we-ended-up-not-using-rails-for-our-new-json-api/
        sensor = create.(params.get(:sensor))

        redirect_to routes.sensor_url(id: sensor.id)
      else
        raise ::ArgumentError, params.errors
      end
    end
  end
end
```

The use of Commands or Interations mixed with the possibility to customize my actions before send it to Repository made me very happy. I made my model responsible only to store state like the [PORO][poro-concept] concept.

##### 5. Umbrella applications

Sometimes we need to create application that shares the same common code. Hanami allows us to create applications inside the our application using the Umbrella concept. This concept allows us to create many applications sharing entities and repositories.

***We know that the set of features that we're going to introduce doesn't belong to our main UI (Web). On the other hand, it's too early for us to implement a microservices architecture, only for the purpose of helping our users reset their password.*** - [Hanami Guide][hanami-architeture]

The Premature Optimization problem can be applied here. Usually working at some startup or a company project wants to be released as fast as we can. Although we prefer to build isolated application, usually we don’t have time enough to create microservices. Using umbrella applications would be a good start to solve this problem.


### The not so bright side

Well, when I was building my first test application (Usgard is my second one), I faced two questions. The first one was *How can I run sidekiq?* It was suffice a quick look at the project issues and I saw my [answer][issue-695], I was using the version 0.9. A little time later, on 1.0.0beta1 it was solved because Hanami added the config/boot.rb as default.

The second one was a question about routes. I have three applications and I mounted to routes file as below.

```ruby
mount Sidekiq::Web,     at: '/sidekiq'
mount Api::Application, at: '/api'
mount Web::Application, at: '/'
```

It was my first contact and I mistakenly changed the order as the following example.

```ruby
mount Web::Application, at: '/'
mount Sidekiq::Web,     at: '/sidekiq'
mount Api::Application, at: '/api'
```
After that, when I visited '/sidekiq' route it redirects to my '/'. I wasted at least two hours to figure out what I did wrong.

Using Hanami maybe you can feel alone or cannot find your answer on internet, in these cases you can go to [Hanami Chat][hanami-chat] to request help or take a look at the [codebase][hanami-codebase].

I think Rails is great and Hanami too. Hanami released the 1.0.0 and a new core team was created then I really believe on this project.

Thanks @Hanamirb and @Rails teams and contribuitors!

### References

* https://docs.spring.io/spring-data/jpa/docs/current/reference/html/
* https://martinfowler.com/eaaCatalog/repository.html
* http://hanamirb.org/guides
* https://12factor.net/config
* http://api.rubyonrails.org/classes/ActionController/Helpers.html
* http://hanamirb.org/guides/architecture/overview/
* https://martinfowler.com/bliki/MonolithFirst.html
* https://github.com/hanami/hanami
* https://mkdev.me/en/posts/a-couple-of-words-about-interactors-in-rails
* https://www.toptal.com/software/single-responsibility-principle
* https://github.com/hanami/view
* https://github.com/hanami/controller
* https://blog.dnsimple.com/2017/03/why-we-ended-up-not-using-rails-for-our-new-json-api/
* http://blog.steveklabnik.com/posts/2011-09-06-the-secret-to-rails-oo-design

[marcello-rocha]: https://twitter.com/mereghost
[poro-concept]: http://blog.steveklabnik.com/posts/2011-09-06-the-secret-to-rails-oo-design
[dnsimple-example]: https://blog.dnsimple.com/2017/03/why-we-ended-up-not-using-rails-for-our-new-json-api/
[hanami-controllers]: https://github.com/hanami/controller
[twelve-factor-config]: https://12factor.net/config
[rails-helpers-doc]: http://api.rubyonrails.org/classes/ActionController/Helpers.html
[toptal-s]: https://www.toptal.com/software/single-responsibility-principle
[hanami-views-doc-description]: https://github.com/hanami/view
[hanami-models-doc]: http://hanamirb.org/guides/models/overview/
[saga-part-one]: http://gabrielmalakias.com.br/hanami/iot/docker/2017/02/14/iot-saga-my-setup-for-a-hanami-application.html
[issue-695]: https://github.com/hanami/hanami/issues/695
[hanami-chat]: https://gitter.im/hanami/chat
[hanami-codebase]: https://github.com/hanami/hanami
[usgard]: https://github.com/GabrielMalakias/usgard
[hanami-architeture]: http://hanamirb.org/guides/architecture/overview/
