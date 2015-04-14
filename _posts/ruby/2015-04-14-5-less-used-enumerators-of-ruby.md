---
layout: post
title:  "5 less used Enumerators of Ruby"
date:   2015-04-14
categories: ruby
tags: ruby
---

This is a repost from [this article](http://codingwithaxe.com/5-less-used-enumerators-of-ruby). It is about 5 less known Enumerators is Ruby.

###Partition
It is used for splitting collection in two parts base on some condition. Example: we want to partition a collection of users in two parts: Admin and Member:

{% highlight ruby %}
class User
  def initialize type
    @type = type
  end

  def is_admin?
    @type == "Admin"
  end

  def is_member?
    @type == "Member"
  end
end

[User.new("Admin"), User.new("Admin"), User.new("Member"),
  User.new("Admin"), User.new("Member")].partition {|u| u.is_admin?}

#=>[[#<User:0x0000000278e2e8 @type="Admin">, #<User:0x0000000278e1f8 @type="Admin">, #<User:0x0000000278de38 @type="Admin">], [#<User:0x0000000278e130 @type="Member">, #<User:0x0000000278dde8 @type="Member">]]
{% endhighlight %}

###Each with object
Use this Enumerator if you want to use the same object for each iteration inside a block. Example:

{% highlight ruby %}
class Person
  def initialize(name)
    @name = name
  end

  def greet(greeter)
    puts "#{greeter} #{@name}"
  end
end

[Person.new("Peter"), Person.new("Meg"), Person.new("Louis")].each_with_object("Hello") { |i, a|  i.greet(a)}
#=>
# Hello Peter
# Hello Meg
# Hello Louis

{% endhighlight %}

###Max By
This method is used to find the object that have the attribute that have the greatest value. Example: we have a list of products and need to return the priciest product in it.

{% highlight ruby %}
class Product
  attr_reader :price

  def initialize(price)
    @price = price
  end
end

[Product.new(100), Product.new(120), Product.new(1000)].max_by(&:price)
=> #<Product:0x007fb019861228 @price=1000>
{% endhighlight %}

###Min Max By
This method returns both the lowest and the highest price products.

{% highlight ruby %}
[Product.new(100), Product.new(120), Product.new(1000)].minmax_by &:price

=> [#<Product:0x007fb01887a418 @price=100>, #<Product:0x007fb01887a3c8 @price=1000>]
{% endhighlight %}

###Take While
If you want to replace while with block style, to build a collection this method is the perfect fit. This method will iterate each element until it meet some condition and return those iterated elements.

{% highlight ruby %}
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10].take_while{|i| i < 8}
# => [1, 2, 3, 4, 5, 6, 7]
{% endhighlight %}
