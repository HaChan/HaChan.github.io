---
layout: post
title:  "Custom Fonts in Rails Asset Pipeline"
date:   2014-10-15
tags: [rails]
---

Rails and the asset pipeline are great tools, but rails has yet worked with custom fonts. But there is a common way in rails to work this thing out.

First, let's consider the rails asset pipeline. In the assets directory, there are 3 folder by default:

```
|-app/
|---assets/
|-----images/
|-----javascripts/
|-----stylesheets/
```

Now, we would want to create a `fonts` directory in assets to store fonts resource for our website.

```ruby
|-app/
|---assets/
|-----fonts/
|-----images/
|-----javascripts/
|-----stylesheets/
```

But the problem is, by default rails asset pipeline only have 3 directory path: images, javascripts, and stylesheets. To enable `fonts` directory in the asset load path, add this configuration in `config/application.rb`:

```ruby
config.assets.paths << Rails.root.join("app", "assets", "fonts")
```

Now, rails asset pipeline is aware of the fonts folder in assets folder.

Default CSS of a custom font look something like this:

```ruby
@font-face {
    font-family: 'icofonts';
    src:url('fonts/icofonts.eot');
    src:url('fonts/icofonts.eot?#iefix') format('embedded-opentype'),
        url('fonts/icofonts.ttf') format('truetype'),
        url('fonts/icofonts.woff') format('woff'),
        url('fonts/icofonts.svg#icofonts') format('svg');
    font-weight: normal;
    font-style: normal;
}
```

This can not work with the asset pipeline. There are two way to make it work. First is replace `src:url()` with `src:font-url()`:

```ruby
@font-face {
  font-family:'icofonts';
  src:font-url('icofonts.eot');
  src:font-url('icofonts.eot?#iefix') format('embedded-opentype'),

  ...
}
```

Second, replace `"fonts/"` with `"/assets/"` inside `src:url`:

```ruby
@font-face {
  font-family: 'icofonts';
  src: url(/assets/icofonts.eot);
  src: url(/assets/icofonts.eot?#iefix) format("embedded-opentype"),

  ...
}
```

[Source](http://anotheruiguy.roughdraft.io/7379570-custom-web-fonts-and-the-rails-asset-pipeline)
