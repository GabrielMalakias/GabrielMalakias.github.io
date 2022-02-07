---
layout: post
title: "Building a simple API using function composition in Ruby"
date:   2022-01-17 00:00:00
published: true
categories: ruby
disqus: true
description: In this post, I will show what is a dependency injection how can we use this concept
---

Function composition was added to Ruby a while a go but I didnt even know about that a month ago. Since I wont use Rails in my next job I decided to group a bunch of gems together and use this concept to write a simple API.

All the code mentioned here can be found [here](https://github.com/GabrielMalakias/composable_web_example).

##### DISCLAIMER
***I used function composition here as a way to challenge myself using building the "business rules" by composing functions. I do not intend to change your mind about the way you should write your apps, the idea here is to experiment and see how far the language allows me to do so.***

### Gluing pieces together

Before starting I defined all the necessities I would have if I where building my own micro web-framework, in the end I came up with the following:

- Server/Routing: [Agoo](https://github.com/ohler55/agoo);
- Database Interface: [Sequel](https://github.com/jeremyevans/sequel);
- Code loader: [Zeitwerk](https://github.com/fxn/zeitwerk);
- Input validations: [Dry-Validation](https://github.com/dry-rb/dry-validation);
- Json serialization: [Oj](https://github.com/dry-rb/dry-validation);

To start the app I defined the following class:

```ruby
# frozen_string_literal: true

# Also available here: https://github.com/GabrielMalakias/composable_web_example/blob/master/application.rb

require 'agoo'
require 'sequel'
require 'zeitwerk'
require 'oj'
require 'dry-validation'

class Application
  DB = Sequel.connect('postgres://localhost:5432', user: 'postgres', password: '123')

  def self.load_files
    loader = Zeitwerk::Loader.new
    loader.push_dir('./lib')
    loader.setup
  end

  def self.start!
    Agoo::Server.init(3000, 'root')
    Agoo::Server.handle(:POST, '/authors', Handler::CreateAuthor.new)
    Agoo::Server.handle(:GET, '/authors/*', Handler::FindAuthor)
    Agoo::Server.start
  end
end

Application.load_files
```

The file above loads the gems needed and:
- Creates the DB connection by defining the `DB` constant managed by `Sequel`;
- A function to load app files within the 'lib' folder lazily using `Zeitwerk`
- A function which initializes and map the routes using `Agoo`;

Besides this file I have also  a `Rakefile` with the following:

```ruby
# frozen_string_literal: true

require 'logger'
require 'sequel'

namespace :app do
  desc 'Start server'
  task :start do
    require_relative 'application'

    Application.start!
  end

  desc 'Open a console'
  task :console do
    exec 'irb -r ./application'
  end
end

namespace :db do
  desc 'Run migrations'
  task :migrate, [:version] do |_t, args|
    require 'sequel/core'
    Sequel.extension :migration

    version = args[:version].to_i if args[:version]

    Sequel.connect('postgres://localhost:5432',
                   user: 'postgres',
                   password: '123',
                   logger: Logger.new($stderr)) do |db|
      Sequel::Migrator.run(db, 'db/migrations', target: version)
    end
  end
end
```

With that Rake can be used for to start the web server, start the console and run migrations.


```sh
master !4 ❯ bundle exec rake -T
rake app:console          # Open a console
rake app:start            # Start server
rake db:migrate[version]  # Run migrations
```

Now that the app is already ready its time to start using the structure it already has.

://wiki.haskell.org/Function_compositions
https://ruby-doc.org/core-2.6/Proc.html#method-i-3C-3C
