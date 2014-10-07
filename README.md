Steps that was done to create this project:

#### Beginning

New rails project from the scratch

`rails new multiple-modules-with-guard`

#### RSpec

Added `rspec-rails` to the `Gemfile`

```ruby
group :test, :development do
  gem 'rspec-rails'
end
```

`rails generate rspec:install`

Notice there is now `rails_helper` and `spec_helper` by default:
* `spec_helper` doesn't load environment (you can require it in tests which doesn't require whole env, to make it run faster)
* `rails_helper` does load environment

#### Model and test

Simple model `Foo`

```ruby
class Foo
end
```

Simple test for `Foo`

```ruby
require 'rails_helper'

describe Foo do
  it { expect(true).to be true }
end
```

#### Guard with spring

Configured `guard-rspec` with `spring`

In the `Gemfile`:
```ruby
group :development do
  gem 'spring'
  gem 'guard-rspec', require: false
  gem 'spring-commands-rspec'
end
```

And in console:

`guard init rspec`

`spring binstub rspec`

Edited generated `Guardfile`
```ruby
guard :rspec, cmd: 'bundle exec spring rspec' do
 # ...
end
```

Sometimes you need to reload `spring` to make them see your binstubs - run then
`spring stop`

#### Module with separate test suite

Now I've added `moduleA` - module with the seperate test suite.

Model (`moduleA/models/a_foo.rb`):

```ruby
class AFoo
end
```

And a test (`moduleA/spec/models/a_foo_spec.rb`):

```ruby
require 'spec_helper'
require './models/a_foo'

describe AFoo do
  it "is true for a AFoo" do
    expect(true).to be false
  end
end
```


I've added the line to monitor it's `.rb` files:

```ruby
watch(%r{^(.+)\.rb$}) { |m| "spec/#{m[1]}_spec.rb" }
```

And I've run guard with `watchdir` option (`-w`) hoping that it will work (run tests for `AFoo` after saving model or test file):

`bundle exec guard start -w . ./moduleA/`

But it won't, so I made some changes to `guard` and `guard-rspec` repos.
So when you include my forks in the `Gemfile`:

```ruby
 gem 'guard', git: 'git@github.com:lesniakania/guard.git', require: false
 gem 'guard-rspec', git: 'git@github.com:lesniakania/guard-rspec.git', require: false
```

And add configuration required for `spring` in `moduleA/config/spring.rb`:

```ruby
Spring.application_root = "../"
```

It works :)
