---
layout: post
title:  "Unary ampersand (&) operator"
date:   2014-10-21
tags: [ruby]
---

The unary & operator is **almost** equivalent of calling `to_proc` on the object. In Ruby, there are two kinds of code blocks: _Block_ and _Proc_. The different between them is that `Proc` can be define (by `Proc.new`, `proc`, `lambda` or `->()`) and reference through variable. While Blocks are always related to a method call and can't be defined anywhere else.

Example:

```ruby
# A block that is passed to the each function on the [1,2,3] Array.
[1,2,3].each do |x|
  puts x
end

# A proc assigned to a variable.
k = Proc.new{ |x| puts x }
```

All method have only one implicit Block argument, and method can access it through `yield`:

```ruby
def two
  yield 2
end

irb(main):001:0> two{|a| a * 3}
=> 6
```

Block can't be defined outside of the function, it will throw syntax error:

```ruby
irb(main):001:0> { |x| x*2 }
=> SyntaxError: syntax error, unexpected '|', expecting '}'
```

While `Proc` can be defined and reference. `Procs` fall into two categories: lambda procs and simple procs. Lambda are defined using `lambda` or `->()`. Simple procs are defined using `Proc.new` or `proc`.

```ruby
irb(main):001:0> lambda{ |x| x*2 }
=> #<Proc:0x00000006598ea8@(pry):3 (lambda)>

irb(main):002:0> ->(x){ x*2 }
=> #<Proc:0x0000000651e6a8@(pry):4 (lambda)>

irb(main):003:0> Proc.new{ |x| x*2 }
=> #<Proc:0x000000064b1bc0@(pry):5>

irb(main):004:0> proc{ |x| x*2 }
=> #<Proc:0x00000006432ac8@(pry):6>
```

The `&` operator is used to switch between Blocks and Procs. `&object` is evaluated in the following way:

  - if object is a block, it is converted into a simple proc.

  - if object is a Proc, it is converted into a block while preserving the `lambda?` status of the object.

  - if object is not a Proc, it first calls `#to_proc` on the object and then converts it into a block.

**If object is a block, it converts the block to a simple proc**

When accessing a block in a method, instead of calling yield, simply use `&` to convert the implicit block to a proc, then it can be accessed through reference in function:

```ruby
def describe &block
  "The block that was passed has parameters: #{block.parameters}"
end

irb(main):001:0> describe{ |a,b| }
=> "The block that was passed has parameters: [[:opt, :a], [:opt, :b]]"

irb(main):002:0> describe do |*args|
irb(main):003:0> end
=> "The block that was passed has parameters: [[:rest, :args]]"
```

**If object is a Proc, it converts the object into a block while preserving the lambda? status of the object.**

This mean lambdas can be passed to a function as block and still preserve their properties like strict argument checking, returning values using `return` keyword.

```ruby
irb(main):001:0> multiply = lambda{ |x| x*2 }

irb(main):002:0> [1,2,3].map(&multiply)
=> [2, 4, 6]
irb(main):003:0> [4,5,6].map(&multiply)
=> [8, 10, 12]
```

**If object is not a Proc, it first calls #to_proc end then converts the object into a block**

This make passing objects to functions in place of blockvery simple. The most common case is probably calling `Array#map` with a symbol.

```ruby
irb(main):001:0> ["1", "2", "3"].map(&:to_i)
=> [1, 2, 3]
```

Because Symbol#to_proc returns a proc that responds to the symbol's method. So the `:to_i` is first converted to a proc, and then it is converted into a block and gets called. So, an object can define it own `to_proc` methods to act with `&` operator.

```ruby
class Display
  def self.to_proc
    lambda{ |x| puts(x) }
  end
end

class FancyDisplay
  def self.to_proc
    lambda{ |x| puts("** #{x} **") }
  end
end


irb(main):001:0> greetings = ["Hi", "Hello", "Welcome"]

irb(main):002:0> greetings.map &Display
Hi
Hello
Welcome
=> [nil, nil, nil]

irb(main):003:0> greetings.map &FancyDisplay
** Hi **
** Hello **
** Welcome **
=> [nil, nil, nil]
```

Source: [The ampersand operator on ruby](http://ablogaboutcode.com/2012/01/04/the-ampersand-operator-in-ruby/)
