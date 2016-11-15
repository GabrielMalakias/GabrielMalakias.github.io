---
layout: post
categories: ruby dry-rb rails
---

Hello everyone, around 2 or 3 months ago i saw dry-rb at first time, i thought: "Oh, that's awesome, i need experiment". Oh, sorry what is Dry-rb? Dry-rb is a bunch of of tools to simplify and improve code. Are TOOLS, how I use? Imagine! Use your creativity, dry-rb one of possible good ways to make some code improvements.

Today in this post i will try to show how use dry-auto_inject with rails. First, what is dry-auto_inject? According with dry-rb it's a "Container-agnostic constructor injection mixin", if you already programmed in languages like Java (Spring @Autowired, says: 'Hello'), or C# or some language or framework with dependency-injection you saw the  amazing 'magic' of Dependency-Injection.

Oh, my Gosh but i never saw nothing about dependency-injection in my life. Oh, relax i dont forgot you. The Internet has a lot materials about dependency injection. If you read about SOLID, the D represents "Dependency inversion principle", this principle can be made with an creational pattern, a factory method or a DEPENDENCY INJECTION framework.


Here is an image to represent this:

![dependency-injection]({{ site.url }}/assets/images/dependency_injection.png)

I will not explain all details behind this because many people already do this as you see below:

* [solnic][solnic-dependency-injection];
* [martin-fowler][martin-fowler-dependency-injection].

If you already saw a code like this:

{% highlight ruby %}
class CreateArticle
  def initialize(repository = ArticleRepository.new)
    this.repository = repository
  end

  def call(article)
    repository.call(article)
  end
end
{% endhighlight %}

In this case the class CreateArticle receives a external dependency and call repository method. CreateArticle believes that ArticleRepository implements a method call and don't have any details about repository parameter. But if you need to change the implementation is necessary change code inside CreateArticle and at place when it's called. If you have some tool to inject this automatically this code can be better.

First you need to install it. I will use current stable rails version 5.0.0.1 and ruby 2.3.0. If you don't have Ruby and Rails installed check how install in [RVM][rvm] or [Rbenv][rbenv] sites, it's very simple to install ;).

{% highlight shell %}
 ~/projects/ruby  ruby -v
ruby 2.3.1p112 (2016-04-26 revision 54768) [x86_64-linux]

~/projects/ruby  rails -v
Rails 5.0.0.1
{% endhighlight %}

After installation, let's begin! Ok, to do the experiments we need an Rails application, to create a new type te command below. if you already has a Rails app, ignore this step,

{% highlight shell %}
~/projects/ruby  rails new blog
create
create  README.md
create  Rakefile
create  config.ru
create  .gitignore
create  Gemfile
create  app
...
{% endhighlight %}

After creation we need to install dry-auto_inject. To install add this line in your Gemfile.

{% highlight ruby %}
gem 'dry-auto_inject'
{% endhighlight %}

{% highlight shell %}
/projects/ruby/blog  bundle list | grep dry
  * dry-auto_inject (0.4.1)
  * dry-configurable (0.1.7)
  * dry-container (0.5.0)
{% endhighlight %}

Dry-auto_inject depends of dry-configurable and dry-container. The dry-configurable is "a simple mixin to add thread-safe configuration behaviour to your classes", you can check here. The dry-container is the core of auto-injection mechanism.

Let's check how it's works.

{% highlight shell %}
~/projects/ruby/blog  bundle exec rails c
Running via Spring preloader in process 5167
Loading development environment (Rails 5.0.0.1)
2.3.1 :001 >
{% endhighlight %}
{% highlight ruby %}
2.3.1 :042 > container = Dry::Container.new
 => #<Dry::Container:0x000000038bb200 @_container={}>
 2.3.1 :043 > container.register(:article_repository) { Class.new }
  => #<Dry::Container:0x000000038bb200 @_container={"article_repository"=>#<Dry::Container::Item:0x000000034d0640 @item=#<Proc:0x000000034d06e0@(irb):43>, @options={:call=>true}>}>
{% endhighlight %}

When somebody calls Dry::Container#register the dependency is stored in a proc, this proc can be called when container resolve the reference or no.

{% highlight ruby %}
2.3.1 :052 > container.register(:hello, ->{ puts "Hello world" }, call: true)
2.3.1 :053 > container.resolve(:hello)
Hello world
 => nil
{% endhighlight %}

And i can store namespaced references.
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

For more details you could check this [documentation][dry-container].

Now, we will create something to use dry-auto_inject. I will use scaffold to generate the Article model.

{% highlight shell %}
 ~/projects/ruby/blog  rails g scaffold Article name:string description:string
 Running via Spring preloader in process 6093
 invoke  active_record
 create    db/migrate/20161115123949_create_articles.rb
 create    app/models/article.rb
 invoke    test_unit
 create      test/models/article_test.rb
 ...

 ~/projects/ruby/blog  rake routes
 Prefix Verb   URI Pattern                  Controller#Action
 articles GET    /articles(.:format)          articles#index
 POST   /articles(.:format)          articles#create
 new_article GET    /articles/new(.:format)      articles#new
 edit_article GET    /articles/:id/edit(.:format) articles#edit
 article GET    /articles/:id(.:format)      articles#show
 PATCH  /articles/:id(.:format)      articles#update
 PUT    /articles/:id(.:format)      articles#update
 DELETE /articles/:id(.:format)      articles#destroy

 ~/projects/ruby/blog  rake db:migrate
  == 20161115123949 CreateArticles: migrating ===================================
-- create_table(:articles)
  -> 0.0015s
  == 20161115123949 CreateArticles: migrated (0.0016s) ==========================

{% endhighlight %}

We will change only the article#create route, but be free to modify anything that you want. Please create an article to test if everything is ok.

After tests, we can change the code. blog/app/controllers/Take a look at articles_controller.rb

{% highlight ruby %}
class ArticlesController < ApplicationController
 ...
  def create
    @article = Article.new(article_params) # we will act only here
    ...
  end
end
{% endhighlight %}

We can create a Command Class only to ilustrate. This command it's very similar concept of [Trailblazer::Operation][trailblazer-operation]. If you don't like, you can use Article.new directly, don't worry :).


Ok, create folder /commands/article under lib directory, after that create create.rb with lines above.

{% highlight ruby %}
#lib/commands/article/create.rb
module Commands
  module Article
    class Create
      def call(params)
        ::Article.new(params).save
      end
    end
  end
end
{% endhighlight %}

If you run rails c and call Commands::Article::Create#call you will get a error, it's because you need to add the line below in Blog::Application, to load folder.

{% highlight ruby %}
#config/application
module Blog
  class Application < Rails::Application
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.
    #
    config.eager_load_paths += Dir["#{Rails.root}/lib"] # add this line
{% endhighlight %}

When you run rails c and call Commands::Article::Create#call, it's works!

{% highlight ruby %}
 ~/projects/ruby/blog  rails c
 Running via Spring preloader in process 10039
 Loading development environment (Rails 5.0.0.1)
 2.3.1 :001 > Commands::Article::Create.new.call(name: "AutoInject", description: "How can i use DryAutoInject")
    (0.1ms)  begin transaction
      SQL (0.5ms)  INSERT INTO "articles" ("name", "description", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["name", "AutoInject"], ["description", "How can i use DryAutoInject"], ["created_at", 2016-11-15 14:03:42 UTC], ["updated_at", 2016-11-15 14:03:42 UTC]]
         (7.9ms)  commit transaction
          => true
{% endhighlight %}

[rvm]: https://rvm.io/rvm/install
[rbenv]: https://github.com/rbenv/rbenv
[dry-container]: http://dry-rb.org/gems/dry-container
[dry-configurable]: http://dry-rb.org/gems/dry-configurable
[trailblazer-operation]: http://trailblazer.to/gems/operation/1.1/
[solnic-dependency-injection]: http://solnic.eu/2013/12/17/the-world-needs-another-post-about-dependency-injection-in-ruby.html
[martin-fowler-dependency-injection]: http://www.martinfowler.com/articles/injection.html
