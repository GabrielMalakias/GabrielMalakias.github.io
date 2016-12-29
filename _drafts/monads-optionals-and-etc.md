---
layout: post
title: 'OO programmers, Monads may be useful to us too'
date:   2016-12-18 00:00:00
categories: ruby java dry-monads optional
---

Hello, I saw the "Monad" concept at first time at TDC(São Paulo) 2016 at the functional programming Track. I didn't get any use possibility in speech, but three months ago, I started to work in a Java project with two collegues, someday a senior job colleague convinced me to use java.util.Optional in some cases along this project "It's cool", he said. He already worked with Scala using some monadics built-in, then he knows the monads advantages.

This project has many sequencial steps, the code majority has to validate something, convert to another type, save in database, send to a queue or something like it. After many sprints interactions, when it started to take form, the code did everything needed and kept all things simple basically composing a lot of Functions applied in Java Optional Objects.

After that, when I came back to Ruby, I started to search some gem with the similar behavior, I googled it and I found Dry-Monads inspired by [Kleisli][kleisli]. When I visited the Kleisli page I saw this phrase "You can use Haskell-like function composition with F and the familiar", I thought "Oh my Gosh, I got Monads now". Moments later, I found a cool post in Quora website about Monads, it's different(and cool) because don't envolves a huge number of mathematical formulas to explain the Monads concept, you can take a look [here][quora], they use Haskell in examples, I don't know Haskell but when if you compare to Dry-monads you will understand too.

#### Intro

Everyone that already programmed in Java, at least once, got an NullPointerException. This error occurs when someone call a function of a null reference.

{% highlight java %}
null.toString();

java.lang.NullPointerException
{% endhighlight %}

If we use Ruby:

{% highlight ruby %}
2.3.1 :004 > nil.to_s
 => ""
 2.3.1 :005 > nil.methods
  => [:&, :^, :|, :===, :inspect, :to_a, :to_s, :to_i, :to_f, :nil?, ... , :instance_eval, :instance_exec, :__id__]

2.3.1 :006 > nil.bla
NoMethodError: undefined method `bla' for nil:NilClass
  from (irb):6
    from /home/gabriel/.rvm/rubies/ruby-2.3.1/bin/irb:11:in `<main>'
{% endhighlight %}

Ruby return an exception when we try to call an inexistent attribute or method, but when we call '.to_s' it returns a blank string. If we use Java, when we call any method it will return an "NullPointerException".

Since Java 8, we have the java.util.Optional, it can be useful to return an empty string when we use a null reference. Let's see the example below:

{% highlight java %}
java.util.Optional.ofNullable(null).map(Object::toString).orElse("");

//And it returns
""
{% endhighlight %}

It's extremely useful to avoid Java NullPointerException. Another cool thing is chain methods, it's possible using some functions as '#map' and '#flatMap'. We can also use the Optional behavior to avoid 'if' sequences and we can to turn our code more readable using the same text way. In Elixir we can chain methods using the pipe operator '\|>', let's check it.

{% highlight elixir %}
"Some String" |> String.upcase

//And it returns
"SOME STRING"
{% endhighlight %}

We can do it in Java too.

{% highlight java %}
java.util.Optional.ofNullable("Some String")
                    .map(String::toUpperCase).get();

//And it returns
"SOME STRING"
{% endhighlight %}


#### Starting

We used this behavior many times in our project than, in this post, I will try show a similar concept using Dry-Monads in a simple 'Order' example.

Ok, let's start, Our very simple use case is:

{% highlight ruby %}
/*
     We have an Order, this Order has a Products, every Product has one price, but it can be nil to gifts, for example. The products sum can't be negative. An Order has a discount. It can be informed or no. Our task is calculate the order total.

     Order
       has -> products
       calc -> total - discount

     Product
       price
*/

{% endhighlight %}


First, we need an Order class

{% highlight ruby %}
class Order
  def total
    @products.pluck(:price).reduce(:+) - discount
  end

  def add_product(product)
    @products = Array(@products) << product
  end

  def discount=(value)
    @discount = value
  end

  def discount
    @discount
  end

  def products=(value)
    @products = value
  end

  def products
    @products
  end
end
{% endhighlight %}

*Ps:. I wrote all getter and setters to turn easier to understand.*

And the Product class

{% highlight ruby %}
class Product
  def initialize(price:)
    @price = price
  end

  def price
    @price
  end
end
{% endhighlight %}

Now, let's check if works creating an Order and some products

{% highlight ruby %}
2.3.1 :001 > order = Order.new
=> #<Order:0x000000031aa0d8>
2.3.1 :002 > products = []
=> []
2.3.1 :003 > 10.times { products << Product.new(price: Random.rand * 10) }
=> 10
2.3.1 :004 > order.products = products
=> [#<Product:0x000000046b2a70 @price=6.490346125032528>, [...More products here], #<Product:0x000000046b24f8 @price=4.088797576323275>, #<Product:0x000000046b2430 @price=5.311290981668898>]
2.3.1 :005 > order.discount= 10
 => 10
 2.3.1 :006 > order.total
  => 34.6909868839012
{% endhighlight %}

Awesome, it works! But if we set discount as nil...

{% highlight ruby %}
2.3.1 :009 > order.discount = nil
 => nil
 2.3.1 :010 > order.total
 TypeError: nil can't be coerced into Float
  from /home/gabriel/projects/ruby/blog/app/models/order.rb:3:in '-'

{% endhighlight %}

Oh :( it's bad. Do you know what happen if at least one product has a nil price?

{% highlight ruby %}
2.3.1 :011 > order.add_product(Product.new(price: nil))
 => [#<Product:0x000000046b2a70 @price=6.490346125032528>, [...More products here], #<Product:0x000000046b2430 @price=5.311290981668898>, #<Product:0x000000045fc108 @price=nil>]
 2.3.1 :012 > order.total
 TypeError: nil can't be coerced into Float
  from /home/gabriel/projects/ruby/blog/app/models/order.rb:3:in '+'
{% endhighlight %}

Is it easy to correct, right? We can add the code below to avoid these exceptions.

{% highlight ruby %}
class Product
...
  def price
    @price || 0
  end
end

class Order
 ...
  def discount
    @discount || 0
  end
end
{% endhighlight %}


{% highlight ruby %}
2.3.1 :001 > order = Order.new
=> #<Order:0x00000004d27a68>
2.3.1 :002 > products = []
=> []
2.3.1 :003 > 10.times { products << Product.new(price: Random.rand * 10) }
=> 10
2.3.1 :004 > order.products = products
=> [#<Product:0x00000004d09248 @price=1.7050223472417392>, [...More products here], #<Product:0x00000004d08b90 @price=1.9066992998913168>, #<Product:0x00000004d08b18 @price=3.178449375930751>]
2.3.1 :005 > order.total
=> 41.55253855960062
2.3.1 :006 > order.add_product(Product.new(price: nil))
=> [#<Product:0x00000004d09248 @price=1.7050223472417392>, [...More products here], #<Product:0x00000004d08b18 @price=3.178449375930751>, #<Product:0x00000004cd1b40 @price=nil>]
2.3.1 :007 > order.total
=> 41.55253855960062
{% endhighlight %}

Magic, it's works! but could we call an order without products?

{% highlight ruby %}
2.3.1 :008 > order.products = nil
 => nil
 2.3.1 :009 > order.total
 NoMethodError: undefined method 'map' for nil:NilClass
{% endhighlight %}

Oh damn it, we can use the '\|\|' operator too or Array(value), but we can try something different using dry-monads. Let's change the code!

{% highlight ruby %}
class Product
  def initialize(price:)
    @price = price
  end

  def price
    Dry::Monads::Maybe(@price).or(Dry::Monads::Some(0))
  end
end

// And the order class

class Order
  def total
    products.bind(-> (products) { products.map { |p| p.price.value }.reduce(:+) - discount.value })
  end

  def add_product(product)
    @products = Array(@products) << product
  end

  def discount=(value)
    @discount = value
  end

  def discount
    Dry::Monads::Maybe(@discount).or(Dry::Monads::Some(0))
  end

  def products=(value)
    @products = value
  end

  def products
    Dry::Monads::Maybe(@products)
  end
end
{% endhighlight %}

Let's try the new version

{% highlight ruby %}
2.3.1 :009 > order = Order.new
=> #<Order:0x00000004d11e70>
2.3.1 :010 > products = []
=> []
2.3.1 :011 > 10.times { products << Product.new(price: Random.rand * 10) }
=> 10
2.3.1 :012 > order.products = products
=> [#<Product:0x00000004cf1788 @price=8.306382864273731>, [...More products here], #<Product:0x00000004cf1080 @price=0.4676475154069548>]
2.3.1 :013 > order.total
=> 38.75444516036125
2.3.1 :014 > order.products = nil
=> nil
2.3.1 :015 > order.total
=> None
{% endhighlight %}

It's works, but the total function isn't readable, we can improve it, but first, take a look:


{% highlight ruby %}
  def total
    products
      .bind(-> (products) { products.map { |p| p.price.value }.reduce(:+) - discount.value })
  end
{% endhighlight %}

The total function has three responsabilities, it collects all products price, sum it and subtract the discount. We can break it into 3 separated functions, something like the code below.
{% highlight ruby %}
class Order
  def total
    products
      .fmap(collect_prices)
      .fmap(sum)
      .bind(subtract_discount)
  end

  def add_product(product)
    @products = Array(@products) << product
  end

  def discount=(value)
    @discount = value
  end

  def discount
    Dry::Monads::Maybe(@discount)
  end

  def products=(value)
    @products = value
  end

  def products
    Dry::Monads::Maybe(@products)
  end

  private

  def collect_prices
    -> (products) { products.map { |p| p.price.value } }
  end

  def sum
    -> (prices) { prices.reduce(:+) }
  end

  def subtract_discount
    ->(total) { discount.fmap(-> (discount) { total - discount }).or(total) }
  end
end

{% endhighlight %}

It's cool, but now we need to add another feature. If our order doesn't has products, we don't need to calculate the total in these cases, then we can add this behavior using the 'Either mixin'. Let's see how we can use it. We will create a new command class 'Commands::Order::Total' and use it to calculate the order price.

{% highlight ruby %}
class Order
  def add_product(product)
    @products = Array(@products) << product
  end

  def discount=(value)
    @discount = value
  end

  def discount
    Dry::Monads::Maybe(@discount)
  end

  def products=(value)
    @products = value
  end

  def products
    Dry::Monads::Maybe(@products)
  end
end

class Commands::Order::Total
  include Dry::Monads::Either::Mixin

  def call(order)
     order.products
      .bind(collect_prices)
      .fmap(sum)
      .bind(subtract_discount(order))
  end

  private

  def collect_prices
    -> (products) {
      return Left('You should add products') if products.empty?
      Right(products.map { |p| p.price.value })
    }
  end

  def sum
    -> (prices) { prices.reduce(:+) }
  end

  def subtract_discount(order)
    ->  (total) { order.discount.fmap(-> (discount) { total - discount }).or(total) }
  end
end


/* Some examples */

2.3.1 :001 > order = Order.new
=> #<Order:0x00000004eea670>
2.3.1 :002 > products = []
=> []
2.3.1 :003 > 10.times { products << Product.new(price: Random.rand * 10) }
=> 10
2.3.1 :004 > calc = Commands::Order::Total.new
=> #<Commands::Order::Total:0x00000004e96c28>
2.3.1 :005 > calc.call order
=> None
2.3.1 :006 > order.products = []
=> []
2.3.1 :007 > calc.call order
=> Left("You should add products to calculate the total")
2.3.1 :008 > order.products = products
=> [#<Product:0x00000004ed4af0 @price=3.8706171589310223>, [...More products here], #<Product:0x00000004ed4668 @price=3.8299514817332634>, #<Product:0x00000004ed45f0 @price=0.595434488114438>]
2.3.1 :009 > calc.call order
=> 37.45434887890415
2.3.1 :010 > order.discount = 10
=> 10
2.3.1 :011 > calc.call order
=> Some(27.45434887890415)
{% endhighlight %}

It works! but the code returns None, Some or Left in some cases. If we use only Left and right we can check if we had success or no. Another thing is the sum of prices can be negative, we can return an failure if it occurs. Let's make some changes.

{% highlight ruby %}
class Commands::Order::Total
  include Dry::Monads::Either::Mixin

  def call(order)
    order.products
    .bind(collect_prices)
    .bind(sum)
    .bind(subtract_discount(order))
  end

  private

  def collect_prices
    -> (products) {
      return Left('You should add products') if products.empty?
      Right(products.map { |p| p.price.value })
    }
  end

  def sum
    -> (prices) {
      total = prices.reduce(:+)
      return Left('The total can\'t be negative') if total < 0
      Right(total)
    }
  end

  def subtract_discount(order)
    ->(total) {
      order.discount
        .bind(-> (discount) { total - discount })
        .bind(-> (total) { Right(total) } )
        .or(Right(total))
    }
  end
end

2.3.1 :039 > order.products = products
=> [#<Product:0x000000043358e8 @price=0.3621462392006869>, [...More products here], #<Product:0x000000044ea0a8 @price=50>, #<Product:0x000000044da630 @price=-120>]
2.3.1 :040 > calc.call order
=> Left("The total can't be negative")
2.3.1 :041 > order.products = []
=> []
2.3.1 :042 > calc.call order
=> Left("You should add products")
2.3.1 :043 > order.add_product(Product.new(price: 300))
=> [#<Product:0x0000000414a088 @price=300>]
  2.3.1 :044 > calc.call order
=> Right(300)
  2.3.1 :045 > order.add_product(Product.new(price: 300))
  => [#<Product:0x0000000414a088 @price=300>, #<Product:0x0000000410ded0 @price=300>]
  2.3.1 :046 > calc.call order
=> Right(600)
2.3.1 :058 > order.discount = 10
 => 10
 2.3.1 :059 > calc.call order
=> Right(590)
2.3.1 :017 > Commands::Order::Total.new.call order
 => Right(51.08628673669591)
2.3.1 :014 > 10.times { products << Product.new({price: Random.rand * 10}) }
 => 10
 2.3.1 :015 > products
 => [#<Product:0x000000052ff4b8 @price=1.8427640303941961>, [...More products here], #<Product:0x000000052feef0 @price=9.708438235799372>]
 2.3.1 :016 > order.products = products
 => [#<Product:0x000000052ff4b8 @price=1.8427640303941961>, [...More products here], #<Product:0x000000052feef0 @price=9.708438235799372>]
2.3.1 :018 > r=_
=> Right(51.08628673669591)
2.3.1 :019 > r.success?
=> true
2.3.1 :020 > products << Product.new({price: -60})
=> [#<Product:0x000000052ff4b8 @price=1.8427640303941961>, [... More products here], #<Product:0x00000005252fd8 @price=-60>]
2.3.1 :021 > order.products = products
=> [#<Product:0x000000052ff4b8 @price=1.8427640303941961>, [...More products here] #<Product:0x00000005252fd8 @price=-60>]
2.3.1 :022 > Commands::Order::Total.new.call order
=> Left("The total can't be negative")
2.3.1 :023 > r=_
=> Left("The total can't be negative")
2.3.1 :024 > r.success?
=> false
{% endhighlight %}

That's all, Is it cool, right? If you like it you can check the page project [here][dry-monads], there are many amazing features.

Thanks!

### References
* http://dry-rb.org/gems/dry-monads/
* https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534#.g2jyjrme3
* https://www.quora.com/What-are-monads-in-functional-programming-and-why-are-they-useful
* https://github.com/txus/kleisli

[kleisli]: https://github.com/txus/kleisli
[dry-monads]: http://dry-rb.org/gems/dry-monads/
[scala-monads]: https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534#.g2jyjrme3
[quora]: https://www.quora.com/What-are-monads-in-functional-programming-and-why-are-they-useful



