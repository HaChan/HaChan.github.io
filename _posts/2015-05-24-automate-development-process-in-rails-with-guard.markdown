---
layout: post
title:  "Automate development process in Rails using Guard"
date:   2015-05-24
tags: [rails]
---

Development process in Rails involving a lot of things like: write test, write code in controllers, models, views... then save it and switch to run the test case that you write before to check if things work. This is just some typical jobs that us Rails developers have to in our daily work. Our jobs might involve other task like writing SCSS and CoffeeScript and then we will have to take several painstaking shell commands (to compile SCSS and CoffeeScript to css and js) to actually make it work. Finally, you probably need to switch to your favourite browser, hit the refresh button to see the page running after you editing your code. The list of works still growing and they are time consuming to manually done by us developer. What we need is a tool to automate these process for us whenever we modify files in our projects.

###What is Guard?

Guard is a file system watcher. It handle events on file system modifications. It detects when you save a file, then runs some commands like run tests in the background, restart the development server, reload the browser, etc.

###Installation

The simplest way to install Guard is to use Bundler.

Add Guard (and any other dependencies) to a Gemfile in your project's root:

```ruby
group :development do
  gem "guard"
end
```

then install with Bundler:

    $ bundle

Generating an empty `Guardfile` in your project's root with:

    $ bundle exec guard init

Running Guard with Bundler:

    $ bundle exec guard

###Installing Guard plugins

Guard alone doesn't do pretty much thing. It only watch for file system change but it doesn't do anything further. To make it does something interesting, you'll need to install its plugins. You can start exploring many Guard plugins available by browsing the [Guard organization](https://github.com/guard) on Github or by searching `guard-` on [Rubygems](https://rubygems.org/search?utf8=%E2%9C%93&query=guard-).

When you have found a Guard plugin you want, add it to your `Gemfile`:

```ruby
group :development do
  gem "guard-<plugin-name>"
end
```

Guard plugin allow us to do some specific tasks like automatically run rspec or Minitest test suit after editing ruby source code, compile CoffeeScript to javascript when a `.coffee` file is changed or automatically refresh browser when some files are changed. It all happen because of these Guard plugins. Lets consider some plugin we need in our daily rails development.

###Rspec Rails

First, to enable Guard to run Rspec tests when the `.rb` files are changed, we need to add `guard-rspec` to our `Gemfile`:

```ruby
group :development do
  gem "guard"
  gem "guard-rspec"
end
```

Now we can run `Bundle` to install it and, once that's done, setup guard with this command:

```
$ guard init rspec
Writing new Guardfile to /path/to/Guardfile
rspec guard added to Guardfile, feel free to edit it
```

This will generate a Guard file for us if it doesn't exist or it would append the configuration for Rspec in the Guardfile. The default of this Guardfile is like so:

```ruby
# A sample Guardfile
# More info at https://github.com/guard/guard#readme

# Note: The cmd option is now required due to the increasing number of ways
#       rspec may be run, below are examples of the most common uses.
#  * bundler: 'bundle exec rspec'
#  * bundler binstubs: 'bin/rspec'
#  * spring: 'bin/rsspec' (This will use spring if running and you have
#                          installed the spring binstubs per the docs)
#  * zeus: 'zeus rspec' (requires the server to be started separetly)
#  * 'just' rspec: 'rspec'
guard :rspec, cmd: 'bundle exec rspec' do
  watch(%r{^spec/.+_spec\.rb$})
  watch(%r{^lib/(.+)\.rb$})     { |m| "spec/lib/#{m[1]}_spec.rb" }
  watch('spec/spec_helper.rb')  { "spec" }

  # Rails example
  watch(%r{^app/(.+)\.rb$})                           { |m| "spec/#{m[1]}_spec.rb" }
  watch(%r{^app/(.*)(\.erb|\.haml|\.slim)$})          { |m| "spec/#{m[1]}#{m[2]}_spec.rb" }
  watch(%r{^app/controllers/(.+)_(controller)\.rb$})  { |m| ["spec/routing/#{m[1]}_routing_spec.rb", "spec/#{m[2]}s/#{m[1]}_#{m[2]}_spec.rb", "spec/acceptance/#{m[1]}_spec.rb"] }
  watch(%r{^spec/support/(.+)\.rb$})                  { "spec" }
  watch('config/routes.rb')                           { "spec/routing" }
  watch('app/controllers/application_controller.rb')  { "spec/controllers" }
  watch('spec/rails_helper.rb')                       { "spec" }

  # Capybara features specs
  watch(%r{^app/views/(.+)/.*\.(erb|haml|slim)$})     { |m| "spec/features/#{m[1]}_spec.rb" }

  # Turnip features and steps
  watch(%r{^spec/acceptance/(.+)\.feature$})
  watch(%r{^spec/acceptance/steps/(.+)_steps\.rb$})   { |m| Dir[File.join("**/#{m[1]}.feature")][0] || 'spec/acceptance' }
end
```

We can now run the `guard` command to start up the Guard server:

    $ guard

If you make a change to your code, for example removing a validation from a model and then save it, Guard will run the Rspec tests immediately and you'll see the broken test.

```ruby
class Post
  extend ActiveModel::Naming
  include ActiveModel::Conversion
  include ActiveModel::Validations

  attr_accessor :blog, :title, :body, :pubdate, :image_url

  #validates :title, presence: true

  def initialize attrs={}
    attrs.each {|k, v| send "#{k}=", v}
  end
end
```

```
Running: spec/models/post_spec.rb
F

Failures:

  1) Post validates title
     Failure/Error: Post.new.should have(1).error_on(:title)
       expected 1 error on :name, got 0
```

**Understanding Guardfile**

You should understand what is written in the Guardfile in order to be able to customizing or maintaining it. Lets consider a line from the Guardfile:

```ruby
watch(%r{^app/(.+)\.rb}) { |m| "spec/#{m[1]}_spec.rb" }))
```

Each line in the Guardfile is a call to a `watch` method with an argument is a regular expression (regex) string and a block which contains the specs file that will be run when a file match the regex from the argument. In the above example, this line match a Ruby source file under the `app/` directory, normally `app/controllers/` and `app/models`, and map to files under `spec` directory.

When the file under the folder e.g `app/models/post.rb` is modified, it's captured by the regex part `%r{^app/(.+)\.rb}` with the 1st group `(.+)`, which is `models/post`, and passed to the block `{ |m| "spec/#{m[1]}_spec.rb" }` with `m[1]` capture the 1st group (`models/post`) from the previous regex. The block then return the string `"spec/models/post.rb"` and the corresponding file will be ran by Rspec.

So, if you want to add some customize matching pattern you should remember that the argument for `wacht` method is for the file pattern that you want `Guard` to watch for modification, and the block is where the file you want `Guard` to invoke by using Rspec commands.

For more options and information, read the [guard-rspec](https://github.com/guard/guard-rspec) on Github.

###CoffeeScript

If you are using CoffeeScript, you will have to do the compiling process to convert CoffeeScript to Javascript. Guard will automate this process for you. First, let's add guard-coffeescript to our Gemfile:

```ruby
group :development do
  #...
  gem "guard-coffeescript"
end
```

Then, initialize Guard for CoffeeScript:

    guard init coffeescript

This will generate or append this contains for your Guardfile:

    guard "coffeescript", :input => "app/assets/javascripts"

We want to keep the CoffeeScript in a different directory than the compiled Javascript, so let's change it to:

    guard "coffeescript", :input => "coffee", :output => "js"

With this configuration, CoffeeScript lives in the `coffee` director and is compiled to the `js` directory.

###Automatically Refresh The Browser with LiveReload

When you're developing webapp, it's common that you make some changes to a file, save, then switch to a web browser and refresh the page to examine changes you make. If you want to save your time from this tedious process then `LiveReload` with `Guard` is the choice for you.

LiveReload is a browser auto-refresh utility. Guard tells LiveReload when files change and LiveReload updates the page automatically.

**Installation**

First, we need to include `guard-livereload` plugin to our `Gemfile`:

```ruby
group :development do
  #...
  gem "guard-livereload"
end
```

When the gem has installed, we can run `guard init livereload` to modify the Guardfile and the start `Guard` again.

Next, we'll need to install a LiveReload extension for Chrome, Safari or Firefox. The [LiveReload instruction page](https://github.com/mockko/livereload/blob/master/README-old.md) has the instruction for downloading the extension for each browser. I'll use the [Chrome Extension](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei) here.

When you have everything installed the job now is just the same with Rspec. In Guardfile, there is a livereload block:

```ruby
guard 'livereload' do
end
```

If you want `Guard` to watch any file modification for LiveReload to automatically refresh the page, simply add a `watch` method line like the Rspec (but without the block because LiveReload will do this for you). Example: if you want LiveReload refresh the browser automatically when a file in `app/views` changed, add this line to the `guard livereload` block:

```ruby
watch(%r{app/.+\.(erb|haml)})
```

Then when you change a file like `app/views/posts/index.html.erb`, the browser will automatically reload the page that has enabled the LiveReload extension.


###Conclusion

`Guard` is a great way to automatically your development process. With enormous Guard plugin, you can automate a lot of process like auto run Rspec or Minitest test when Ruby files change, compile CoffeeScript and SASS files when they change, automatically run the `bundle` command when the `Gemfile` changes, restart a Passenger or Pow server when a config or initializer file changes or even auto refreshing the browser when certain file change. Guard is a great tool to increase your productivity.

References: [railscasts](http://railscasts.com/episodes/264-guard?view=asciicast), [sitepoint](http://www.sitepoint.com/automatically-reload-things-guard/)
