---
layout: post
title: 'Dry-Monads(Ruby) and Optional(Java) may be useful'
date:   2016-12-18 00:00:00
categories: ruby java dry-monads optional
---

Hello, In this post we will try to use a gem called "dry-monads". I saw the "Monad" concept at first time at TDC(São Paulo) 2016 at the functional programming Track. I didn't get any possibility in speech, but after that, when I started to work in a Java project, I used java.util.Optional a lot and after that, I came back to Ruby and start to search some gem with the similar behavior and I found Dry-Monads inspired by Kleisli. When I visited the Kleisli page I saw this phrase "You can use Haskell-like function composition with F and the familiar", I thought "Oh my Gosh, I got it now". I found a cool post in Quora website you can take a look [here][quora], they use Haskell in examples, I don't know Haskell but when I compared with Dry-monads I understood and I liked it.

**DISCLAIMER: Is not my objective talk about what language is better because it's a personal opinion and every language has an existence motivation. You can use any language, if it attends your objective or already has implemented code using it, from my point of view, it's the best option to you.**

Everyone that already programmed in Java, at least once, got an NullPointerException. This error it's caused when someone calls something using a null object.

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

Ruby return an exception when I try to call an inexistent attribute or method, but when we call '.to_s' it returns an blank string. If we use Java when we call any method it will return an "NullPointerException".

At since of Java 8, we have the java.util.Optional, it can be useful to our case to return an empty string when it's null. Let's see the example below:

{% highlight java %}
java.util.Optional.ofNullable(null).map(Object::toString).orElse("");

//And it returns
""
{% endhighlight %}

It's extremely useful to avoid Java NullPointerException and to chain methods. We also can use the Optional behavior to avoid 'if' sequences and we can to turn our code more readable using the same form that we write texts. Elixir has a similar behavior to "map", the pipe operator '\|>' also can be used to chain methods, let's check it.

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

In our new company project we used a lot the java.util.Optional, the code still readable and simple. In this post I will try to use Dry-Monads in a simple 'Order' example.

Ok, let's start, Our very simple use case is:

{% highlight ruby %}
/*
  We have an Order, this Order has a Products, every Product has one price, it can be nil. The discount can be informed it be nil too. Our task is calculate the total at the final.

  Order
    has -> products
    calc -> total - discount

  Product
    price
*/
{% endhighlight %}


We need an Order class

{% highlight ruby %}
class Order
  def total
    @products.pluck(:price).reduce(:+) - discount
  end

  def add_product(product)
    @products = Array(products) << product
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

Awesome, it works! But what happens if we set discount as nil?

{% highlight ruby %}
2.3.1 :009 > order.discount = nil
 => nil
 2.3.1 :010 > order.total
 TypeError: nil can't be coerced into Float
  from /home/gabriel/projects/ruby/blog/app/models/order.rb:3:in '-'

{% endhighlight %}

Oh :( it's bad and if any product has a nil price?

{% highlight ruby %}
2.3.1 :011 > order.add_product(Product.new(price: nil))
 => [#<Product:0x000000046b2a70 @price=6.490346125032528>, [...More products here], #<Product:0x000000046b2430 @price=5.311290981668898>, #<Product:0x000000045fc108 @price=nil>]
 2.3.1 :012 > order.total
 TypeError: nil can't be coerced into Float
  from /home/gabriel/projects/ruby/blog/app/models/order.rb:3:in '+'
{% endhighlight %}

Is it easy to correct right? We can add the code below to avoid these exceptions.

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

Magic, it's works! but if we call an order without products?

{% highlight ruby %}
2.3.1 :008 > order.products = nil
 => nil
 2.3.1 :009 > order.total
 NoMethodError: undefined method 'map' for nil:NilClass
{% endhighlight %}

Oh damn it, we can use the '\|\|' operator too, but we can try something different using dry-monads. Let's change the code!

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
    @products = Array(products) << product
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

It's works, but the total function isn't readable, this function is unclear, we can improve it, but first take a look:


{% highlight ruby %}
  def total
    products
      .bind(-> (products) { products.map { |p| p.price.value }.reduce(:+) - discount.value })
  end
{% endhighlight %}

The total function has three responsabilities, it collects products price, sum all and subtract the discount. We can break it in 3 separated functions, something like the code below.
{% highlight ruby %}

class Order
  def total
    products
      .fmap(-> (products) { collect_prices(products) })
      .fmap(-> (prices) { sum(prices) })
      .bind(-> (total) { subtract_discount(total) })
  end

  def add_product(product)
    @products = Array(products) << product
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

  def collect_prices(products)
    products.map { |p| p.price.value }
  end

  def sum(prices)
    prices.reduce(:+)
  end

  def subtract_discount(total)
    total - discount.value
  end
end
{% endhighlight %}



[quora]: https://www.quora.com/What-are-monads-in-functional-programming-and-why-are-they-useful

