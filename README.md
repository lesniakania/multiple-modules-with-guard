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

#### Using `chdir` guard option in the Guardfile

```ruby
[['.', ''], ['moduleA', 'moduleA/']].each do |dir, prefix|
  guard :rspec,
  chdir: dir,
  cmd: "cd #{dir} && bundle exec spring rspec" do

  watch(%r{^#{prefix}(.+)\.rb$})
  watch(%r{^#{prefix}lib/(.+)\.rb$}) { |m| "spec/lib/#{m[1]}_spec.rb" }
  watch("#{prefix}spec/spec_helper.rb") { "spec" }

  # Rails example
  watch(%r{^#{prefix}app/(.+)\.rb$}) { |m| "spec/#{m[1]}_spec.rb" }
  watch(%r{^#{prefix}(.+)\.rb$}) { |m| "spec/#{m[1]}_spec.rb" }
  watch(%r{^#{prefix}app/(.*)(\.erb|\.haml|\.slim)$}) { |m| "spec/#{m[1]}#{m[2]}_spec.rb" }
  watch(%r{^#{prefix}app/controllers/(.+)_(controller)\.rb$}) { |m| ["spec/routing/#{m[1]}_routing_spec.rb", "spec/#{m[2]}s/#{m[1]}_#{m[2]}_spec.rb", "spec/acceptance/#{m[1]}_spec.rb"] }
  watch(%r{^#{prefix}spec/support/(.+)\.rb$}) { "spec" }
  watch("#{prefix}config/routes.rb") { "spec/routing" }
  watch("#{prefix}app/controllers/application_controller.rb") { "spec/controllers" }
  watch("#{prefix}spec/rails_helper.rb") { "spec" }

  # Capybara features specs
  watch(%r{^#{prefix}app/views/(.+)/.*\.(erb|haml|slim)$}) { |m| "spec/features/#{m[1]}_spec.rb" }

  # Turnip features and steps
  watch(%r{^#{prefix}spec/acceptance/(.+)\.feature$})
  watch(%r{^#{prefix}spec/acceptance/steps/(.+)_steps\.rb$}) { |m| Dir[File.join("**/#{m[1]}.feature")][0] || 'spec/acceptance' }
end
```

There are 4 important things there:
- `chdir` option passed to `guard` method
- change in `cmd` - `guard` needs to change directory to execute your tests in `moduleA`
- `prefix` needs to be added to every pattern
- everything is done in the loop for every test suite

New option is not released yet, so for now you need to use `guard` gems from the Github repo.

```ruby
gem 'guard', git: 'git@github.com:guard/guard.git'
gem 'guard-rspec', git: 'git@github.com:guard/guard-rspec.git'
```

And add configuration required for `spring` in `moduleA/config/spring.rb`:

```ruby
Spring.application_root = "../"
```

It works :)
