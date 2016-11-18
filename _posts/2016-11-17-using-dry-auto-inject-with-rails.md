---
layout: post
title: "Using dry-auto_inject with rails"
date:   2016-11-17 12:00:00
categories: ruby dry-rb rails
disqus: true
---

Around of 2 or 3 months ago, I saw dry-rb at the first time, I thought: "Oh, that's awesome, I need experiment". Today is the day! If you are thinking: "Oh, sorry what is Dry-rb?". I will explain... Well, Dry-rb is a bunch of tools to simplify and sometimes to do some code improvements.

Today in this post, I will try to show how we can use dry-auto_inject with rails. First, what is dry-auto_inject? According with dry-rb.org, it's a "Container-agnostic constructor injection mixin", if you already programmed in languages like Java (Spring @Autowired, says: 'Hello'), C#, or some language or framework with dependency-injection support you saw the amazing 'magic' of Dependency Injection(DI).

If you are thinking: "Oh, my Gosh but I never saw nothing about dependency injection in my life". Relax, I didn't forget you. The Internet has a lot materials about dependency injection. If you read something about SOLID, you already saw this concept. The D in SOLID, represents "Dependency inversion principle", this principle can be made with an creational pattern, a factory method or a DEPENDENCY INJECTION framework.

You can see below an image to represent DI:

![dependency-injection]({{ site.url }}/assets/images/dependency_injection.png)

I will not explain all details behind this concept, because many people already do it as you see below:

* [solnic][solnic-dependency-injection];
* [martin-fowler][martin-fowler-dependency-injection].

But, if you already saw something like the code below, you already had a possibility to use DI:

{% highlight ruby %}
class CreateArticle
  def initialize(repository = ArticleRepository.new)
    self.repository = repository
  end

  def call(article)
    repository.call(article)
  end
end
{% endhighlight %}

In this case the class CreateArticle receives a external dependency and call repository method. The class CreateArticle believes that ArticleRepository implements a method #call, it doesn't have any details about repository. If you need to change the code implementation, you need to do changes only inside ArticleRepository. If you have some tool to inject the dependency automatically, this code can be more uncoupled and the responsability to know who implements, can be delegated to another part of code.

#### Starting
First, we need to install the development environment. We will use current stable rails version 5.0.0.1 and ruby 2.3.0. If you don't have Ruby and Rails installed, check how install in [RVM][rvm] or [Rbenv][rbenv] sites, it's very simple ;).

{% highlight shell %}
 ~/projects/ruby  ruby -v
ruby 2.3.1p112 (2016-04-26 revision 54768) [x86_64-linux]

~/projects/ruby  rails -v
Rails 5.0.0.1
{% endhighlight %}

After that, we can start! To do the experiments, we need a Rails application. To create a new App, type the command below. If you already has a Rails app, you can ignore this step.

{% highlight shell %}
~/projects/ruby  rails new blog
create
create  README.md
create  Rakefile
create  config.ru
create  .gitignore
create  Gemfile
create  app
...
{% endhighlight %}

After create app, we need to install dry-auto_inject. To install, add this line in your Gemfile.

{% highlight ruby %}
gem 'dry-auto_inject'
{% endhighlight %}

And run

{% highlight shell %}
bundle install
{% endhighlight %}

{% highlight shell %}
/projects/ruby/blog  bundle list | grep dry
  * dry-auto_inject (0.4.1)
  * dry-configurable (0.1.7)
  * dry-container (0.5.0)
{% endhighlight %}

Dry-auto_inject depends of dry-configurable and dry-container. The dry-configurable is "a simple mixin to add thread-safe configuration behaviour to your classes", you can check [here][dry-configurable]. The dry-container is the core of auto-injection mechanism.

Let's see how it works.

{% highlight shell %}
~/projects/ruby/blog  bundle exec rails c
Running via Spring preloader in process 5167
Loading development environment (Rails 5.0.0.1)
2.3.1 :001 >
{% endhighlight %}
{% highlight ruby %}
2.3.1 :002 > container = Dry::Container.new
 => #<Dry::Container:0x000000038bb200 @_container={}>
 2.3.1 :003 > container.register(:article_repository) { Class.new }
  => #<Dry::Container:0x000000038bb200 @_container={"article_repository"=>#<Dry::Container::Item:0x000000034d0640 @item=#<Proc:0x000000034d06e0@(irb):43>, @options={:call=>true}>}>
{% endhighlight %}

When someone call Dry::Container#register with the identifier(:article_repository), the dependency is stored inside a proc. After that, when someone call Dry::Container#resolve with identifier(:article_repository) the proc stored previosly is executed or returned. The code inside proc can be executed immediately when resolve the reference or no, let's see how it works with the code below.

{% highlight ruby %}
2.3.1 :001 > container = Dry::Container.new
=> #<Dry::Container:0x000000042db460 @_container={}>
2.3.1 :002 > container.register(:lazy_hello, -> { puts "Hello Lazy" }, call: false)
=> #<Dry::Container:0x000000042db460 @_container={"lazy_hello"=>#<Dry::Container::Item:0x00000004342a98 @item=#<Proc:0x000000043432e0@(irb):2 (lambda)>, @options={:call=>false}>}>
2.3.1 :003 > container.resolve(:lazy_hello)
=> #<Proc:0x000000043432e0@(irb):2 (lambda)>
2.3.1 :004 > container.resolve(:lazy_hello).call
Hello Lazy
=> nil
2.3.1 :005 > container.register(:hello, -> { puts "Hello" }, call: true)
=> #<Dry::Container:0x000000042db460 @_container={"lazy_hello"=>#<Dry::Container::Item:0x00000004342a98 @item=#<Proc:0x000000043432e0@(irb):2 (lambda)>, @options={:call=>false}>, "hello"=>#<Dry::Container::Item:0x000000043814f0 @item=#<Proc:0x000000043815b8@(irb):5 (lambda)>, @options={:call=>true}>}>
2.3.1 :006 > container.resolve(:hello)
Hello
=> nil
{% endhighlight %}

And we can store namespaced identifiers.
{% highlight ruby %}
2.3.1 :060 > container = Dry::Container.new
 => #<Dry::Container:0x00000003cebc08 @_container={}>
 2.3.1 :061 > container.namespace(:services) do
 2.3.1 :062 >     namespace(:article) do
 2.3.1 :063 >       register(:repository) { Class.new }
 2.3.1 :064?>     end
 2.3.1 :065?>   end
  => #<Dry::Container:0x00000003cebc08 @_container={"services.article.repository"=>#<Dry::Container::Item:0x00000003c898a0 @item=#<Proc:0x00000003c89ad0@(irb):63>, @options={:call=>true}>}>
  2.3.1 :066 > container.resolve('services.article.repository')
   => #<Class:0x00000003c66490>
{% endhighlight %}

For more details, you could check this [documentation][dry-container].

Now, we need to create something to use dry-auto_inject. We will use scaffold to generate the Article model.

{% highlight shell %}
 ~/projects/ruby/blog  rails g scaffold Article name:string description:string
 Running via Spring preloader in process 6093
 invoke  active_record
 create    db/migrate/20161115123949_create_articles.rb
 create    app/models/article.rb
 invoke    test_unit
 create      test/models/article_test.rb
 ...

{% endhighlight %}

Now, we can see the created routes typing 'rake routes'.

{% highlight shell %}
 ~/projects/ruby/blog  rake routes
 Prefix Verb   URI Pattern                  Controller#Action
 articles GET    /articles(.:format)          articles#index
 POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
 edit_article GET    /articles/:id/edit(.:format) articles#edit
 article GET    /articles/:id(.:format)      articles#show
 PATCH  /articles/:id(.:format)      articles#update
 PUT    /articles/:id(.:format)      articles#update
 DELETE /articles/:id(.:format)      articles#destroy
{% endhighlight %}

Before run application, we need to create the table to store all article entries.

{% highlight shell %}
 ~/projects/ruby/blog  rake routes
 ~/projects/ruby/blog  rake db:migrate
  == 20161115123949 CreateArticles: migrating ===================================
-- create_table(:articles)
  -> 0.0015s
  == 20161115123949 CreateArticles: migrated (0.0016s) ==========================
{% endhighlight %}

We will change only the article#create route, but be free to modify everything you want. You can create an article to test if everything work as expected, to do this, open your browser and access http://localhost:3000/articles.

After tests, we can change the code. Take a look at articles_controller.rb.

{% highlight ruby %}
#app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
 ...
  def create
    @article = Article.new(article_params) # we will act only here
    ...
  end
end
{% endhighlight %}

I will create a command class only for illustration proposes. The command concept here, is very similar of [Trailblazer::Operation][trailblazer-operation]. If you don't like it, you can use Article#create to create directly, don't worry :).

Now, create the folder /commands/article under lib directory, after that, create file create.rb with lines below.

{% highlight ruby %}
#lib/commands/article/create.rb
module Commands
  module Article
    class Create
      def call(params)
        ::Article.create(params)
      end
    end
  end
end
{% endhighlight %}

If you run rails console and call Commands::Article::Create#call you will get an error, it's because we need to add the line responsible to load the created structure at Blog::Application class.

{% highlight ruby %}
#config/application
module Blog
  class Application < Rails::Application
    config.eager_load_paths += Dir["#{Rails.root}/lib"] # add this line
{% endhighlight %}

Now you can run rails console and call Commands::Article::Create#call with params. An article should be created.

{% highlight ruby %}
 ~/projects/ruby/blog  rails console
 Running via Spring preloader in process 10039
 Loading development environment (Rails 5.0.0.1)
 2.3.1 :001 > Commands::Article::Create.new.call(name: "AutoInject", description: "How can i use DryAutoInject")
    (0.1ms)  begin transaction
      SQL (0.5ms)  INSERT INTO "articles" ("name", "description", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["name", "AutoInject"], ["description", "How can i use DryAutoInject"], ["created_at", 2016-11-15 14:03:42 UTC], ["updated_at", 2016-11-15 14:03:42 UTC]]
         (7.9ms)  commit transaction
          => true
{% endhighlight %}

Ok, after check we can start to add dry-auto_inject. First, we need to define the container to register all dependencies needed. To do this task, we need to create a file under config/initializers to register all dependencies.

{% highlight ruby %}
#config/initializers/auto_inject.rb
class Blog::Container
  extend Dry::Container::Mixin

  register('commands.article.create') do
    Commands::Article::Create.new
  end
end

AutoInject = Dry::AutoInject(Blog::Container)
{% endhighlight %}

At the last line of file, the constant AutoInject was defined, this constant will be used inside of ArticlesController to inject dependencies.

To use the registered command, we need to include a reference to registered dependency. The code will be like this:

{% highlight ruby %}
#app/controllers/articles_controller.rb
class ArticlesController < ApplicationController
  # The alias create_article it's a reference
  # to 'commands.article.create'
  include AutoInject[create_article: 'commands.article.create']
  ...

  def create
    @article = create_article.call(article_params)

    respond_to do |format|
      if @article.persisted?
{% endhighlight %}

Finally, you can test again to see DI working. To see how add tests and many cool things please check the [site][dry-rb].

All code is available on my [github][dry-experiences].

Thanks!

#### References
* http://dry-rb.org/;
* http://trailblazer.to/gems/operation/1.1/;
* http://www.martinfowler.com/articles/injection.html;
* http://solnic.eu/2013/12/17/the-world-needs-another-post-about-dependency-injection-in-ruby.html;
* https://en.wikipedia.org/wiki/Dependency_inversion_principle.

[rvm]: https://rvm.io/rvm/install
[rbenv]: https://github.com/rbenv/rbenv
[dry-rb]: http://dry-rb.org/
[dry-container]: http://dry-rb.org/gems/dry-container
[dry-experiences]: https://github.com/GabrielMalakias/dry-experiences
[dry-configurable]: http://dry-rb.org/gems/dry-configurable
[trailblazer-operation]: http://trailblazer.to/gems/operation/1.1/
[solnic-dependency-injection]: http://solnic.eu/2013/12/17/the-world-needs-another-post-about-dependency-injection-in-ruby.html
[martin-fowler-dependency-injection]: http://www.martinfowler.com/articles/injection.html
