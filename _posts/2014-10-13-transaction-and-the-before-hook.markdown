---
layout: post
title:  "Transaction and the before hook"
date:   2014-10-13
tags: [ruby,rspec]
---

Today I have learn a new thing about the before hook and Transaction in RSpec. The default configuration of RSpec is:

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = true
end
```

What this mean in Rails is that _"it run every test/example within a transaction"_. The idea is to start each example with a clean database, create any necessary data for the example and then remove that data by rolling back at the end of that test/example for isolating test.

The data created in `before(:each)` are rolled back by this setting but the data created in `before(:all)` are not rolled back. To clean the data created by `before(:all)`, you have to do it in the `after(:all)` hook:

```ruby
before(:all) do
  @widget = Widget.create!
end

after(:all) do
  @widget.destroy
end
```

And reload it in the before(:each) hook:

```ruby
before(:all) do
  @widget = Widget.create!
end

before(:each) do
  @widget.reload
end
```

To manually manage the data, simply set:

```ruby
config.use_transactional_fixtures = false
```

Data that created within a transaction is invisible to other transaction. This cause proble when using Capybara with javascript test, since the test suite and the underlying browser server are running in different thread, therefore data created in the test suite is invisible to the browser.

So, in order to keep it transactionable, use database_cleaner with these config:

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = false

  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
end
```

These configuration instruct database_cleaner to still use transaction for normal RSpec test, and for the test involve javascript, use trucation instead.

To clean data created by before(:all), use database_cleaner with this option:

```ruby
after(:all) do
  DatabaseCleaner.clean_with(:truncation)
end
```

Source: [Rspec doc](https://www.relishapp.com/rspec/rspec-rails/docs/transactions)
