---
layout: post
title:  "Splat for concatenation. Clean up custom rails helpers with concat and capture"
date:   2014-10-14
tags: [ruby,rails]
---

###Splat for concatenation

In Ruby, the splat operator (*) is used for any-length argument lists:

```ruby
def make_into_array(*args)
  args
end

make_into_array(1, 2)  # => [ 1, 2 ]
```

It can be used as a list flattener, though.

You can use it for nicer array concatenation, like so:

```ruby
foos = [ 1, 2 ]

[ *foos, "more" ]
# => [ 1, 2, "more" ]

[ "more", *foos, *foos, "even more" ]
# => [ "more", 1, 2, 1, 2, "even more" ]
```

It can also be used for argument concatenation:

```ruby
COMMON_ACCESSIBLE = [ :name, :email ]
attr_accessible *COMMON_ACCESSIBLE, as: :user

attr_accessible *COMMON_ACCESSIBLE, :is_admin, as: :admin
```

Basically, splat lets array or argument list can be put anywhere inside another array or argument list in a "flat" way - making the element part of that list itself.

###Clean up custom rails helpers with concat and capture

A normal way to concatenate built-in rails helpers, such as `content_tag` or `link_to` is using `+` or `<<`:

```ruby
module MyHelper
  def widget
    content_tag(:p, class: "widget") do
      link_to("Hello", hello_path) << " " << link_to("Bye", goodbye_path)
    end
  end
end
```

Rails provide a handy function concat to concatenate those helpers:

```ruby
module MyHelper
  def widget
    content_tag(:p, class: "widget") do
      concat link_to("Hello", hello_path)
      concat " "
      concat link_to("Bye", goodbye_path)
    end
  end
end
```

The `concat` method is used to output text in a non-output block (i.e <% %>),  similar to the output block (<%= %>):

```ruby
<%
concat "hello"
# is the equivalent of <%= "hello" %>
%>
```

When using `concat` outside a block helper (e.g `link_to` or `content_tag`), because the behavior of concat, it can be output directly in a non-output block. Thus, when using it with output block, it will render twice. Example with `widget` method from `MyHelper` module from the previous example:

```ruby
<% widget %> #This will render

<%= widget %> #This will render tiwce
```

This is because, inside a block helper of an element, concat appends only to that element's content. But when using it outside, `concat` will return and append directly to the content.

In order to use `concat` without output directly to the output buffer, simply wrap it with a helper block, which does output only concatenated thing inside it. And that what `capture` helper does:

```ruby
module MyHelper
  def widget
    capture do
      concat link_to("Hello", hello_path)
      concat " "
      concat link_to("Bye", goodbye_path)
    end
  end
end
```

This technique is very crucial when rendering a complex helper with a lot of logic influencing the choice of elements or attributes.

Sources: [thepugautomatic](http://thepugautomatic.com/2013/06/helpers/), [api rubyonrails](http://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html#method-i-concat)
