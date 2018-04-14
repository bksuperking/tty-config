# TTY::Config [![Gitter](https://badges.gitter.im/Join%20Chat.svg)][gitter]

[![Gem Version](https://badge.fury.io/rb/tty-config.svg)][gem]
[![Build Status](https://secure.travis-ci.org/piotrmurach/tty-config.svg?branch=master)][travis]
[![Build status](https://ci.appveyor.com/api/projects/status/2383i0dn3hlw9cnn?svg=true)][appveyor]
[![Maintainability](https://api.codeclimate.com/v1/badges/dfac05073e1549e9dbb6/maintainability)][codeclimate]
[![Coverage Status](https://coveralls.io/repos/github/piotrmurach/tty-config/badge.svg)][coverage]
[![Inline docs](http://inch-ci.org/github/piotrmurach/tty-config.svg?branch=master)][inchpages]

[gitter]: https://gitter.im/piotrmurach/tty
[gem]: http://badge.fury.io/rb/tty-config
[travis]: http://travis-ci.org/piotrmurach/tty-config
[appveyor]: https://ci.appveyor.com/project/piotrmurach/tty-config
[codeclimate]: https://codeclimate.com/github/piotrmurach/tty-config/maintainability
[coverage]: https://coveralls.io/github/piotrmurach/tty-config
[inchpages]: http://inch-ci.org/github/piotrmurach/tty-config

> Define, read and write any Ruby app configurations with a penchant for terminal clients.

**TTY::Config** provides app configuration component for [TTY](https://github.com/piotrmurach/tty) toolkit.

## Features

* Read & write configurations in YAML, JSON, TOML and other formats
* Simple interface for setting and fetching values for deeply nested keys
* Merging of configuration options from other hashes

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'tty-config'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install tty-config

## Contents

* [1. Usage](#1-usage)
* [2. Interface](#2-interface)
  * [2.1 set](#21-set)
  * [2.2 set_if_empty](#22-set_if_empty)
  * [2.3 fetch](#23-fetch)
  * [2.4 merge](#24-merge)
  * [2.5 append](#25-append)
  * [2.6 remove](#26-remove)
  * [2.7 delete](#27-delete)
  * [2.8 filename=](#28-filename=)
  * [2.9 append_path/prepend_path](#29-append_pathprepend_path)
  * [2.10 read](#210-read)
  * [2.11 write](#211-write)

## 1. Usage

Initialize the configuration and provide the name:

```ruby
config = TTY::Config.new
config.filename = 'investments'
```

then configure values for different nested keys with `set` and `append`:

```ruby
config.set(:settings, :base, value: 'USD')
config.set(:settings, :color, value: true)
config.set(:coins, value: ['BTC'])

config.append('ETH', 'TRX', 'DASH', to: :coins)
```

get any value by using `fetch`:

```ruby
config.fetch(:settings, :base)
# => 'USD'

config.fetch(:coins)
# => ['BTC', 'ETH', 'TRX', 'DASH']
```

and `write` configration out to `investments.yml`:

```ruby
config.write
# =>
# ---
# settings:
#   base: USD
#   color: true
# coins:
#  - BTC
#  - ETH
#  - TRX
#  - DASH
```

and then to read an `investments.yml` file, you need to provide the locations to search in:

```ruby
config.append_path Dir.pwd
config.append_path Dir.home
```

Finally, read in configuration back again:

```ruby
config.read
```

## 2. Interface

### 2.1 set

To set configuration setting use `set` method. It accepts any number of keys and value by either using `:value` keyword argument or passing a block:

```ruby
config.set(:base, value: 'USD')
config.set(:base) { 'USD' }
```

The block version of specifying a value will mean that the value is evaluated every time its being read.

You can also specify deeply nested configuration settings by passing sequence of keys:

```ruby
config.set :settings, :base, value: 'USD'
```

is equivalent to:

```ruby
config.set 'settings.base', value: 'USD'
```

### 2.2 set_if_empty

To set a configuration setting only if it hasn't been set before use `set_if_empty`:

```ruby
config.set_if_empty :base, value: 'USD'
```

Similar to `set` it allows you to specify arbitrary sequence of keys followed by a key value or block:

```ruby
config.set_if_empty :settings, :base, value: 'USD'
```

### 2.3 fetch

To get a configuration setting use `fetch`, which can accept default value either with a `:default` keyword or a block that will be lazy evaluated:

```ruby
config.fetch(:base, default: 'USD')
config.fetch(:base) { 'USD' }
```

Similar to `set` operation, `fetch` allows you to retrieve deeply nested values:

```ruby
config.fetch(:settings, :base) # => USD
```

is equivalent to:

```ruby
config.fetch('settings.base')
```

`fetch` has indifferent access so you can mix string and symbol keys, all the following examples retrieve the value:

```ruby
config.fetch(:settings, :base)
config.fetch('settings', 'base')
config.fetch(:settings', 'base')
config.fetch('settings', :base)
```

### 2.4 merge

To merge in other configuration settings as hash use `merge`:

```ruby
config.set(:a, :b, value: 1)
config.set(:a, :c, value: 2)

config.merge({:a => {:c => 3, d: => 4}})

config.fetch(:a, :c) # => 3
```

### 2.5 append

To append arbitrary number of values to a value under a given key use `append`:

```ruby
config.set(:coins) { ["BTC"] }

config.append("ETH", "TRX", to: :coins)
# =>
# {coins: ["BTC", "ETH", "TRX"]}
```

You can also append values to deeply nested keys:

```ruby
config.set(:settings, :bases, value: ["USD"])

config.append("EUR", "GBP", to: [:settings, :bases])
# =>
# {settings: {bases: ["USD", "EUR", "GBP"]}}
```

### 2.6 remove

Use `remove` to remove a set of values from a key.

```ruby
config.set(:coins, value: ["BTC", "TRX", "ETH", "DASH"])

config.remove("TRX", "DASH", from: :coins)
# =>
# ["BTC", "ETH"]
```

If the key is nested the `:from` accepts an array:

```ruby
config.set(:holdings, :coins, value: ["BTC", "TRX", "ETH", "DASH"])

config.remove("TRX", "DASH", from: [:holdings, :coins])
# =>
# ["BTC", "ETH"]
```

### 2.7 delete

To completely delete a value and corresponding key use `delete`:

```ruby
config.set(:base, "USD")
config.delete(:base)
# =>
# "USD"
```

You can also delete deeply nested keys and their values:

```ruby
config.set(:settings, :base, "USD")
config.delete(:settings, :base)
# =>
# "USD"
```

### 2.8 filename

### 2.9 append_path/prepend_path

You need to tell the **TTY::Config** where to search for configuration files. To search multiple paths for a configuration file use `append_path` or `prepend_path` methods.

For example, if you want to search through `/etc` directory first, then user home directory and then current directory do:

```ruby
config.append_path("/etc/")   # look in /etc directory
config.append_path(Dir.home)  # look in user's home directory
config.append_path(Dir.pwd)   # look in current working directory
```

None of these paths are required, but you should provide at least one path if you wish to read configuration file.

### 2.10 read


### 2.11 write

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/tty-config. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the Tty::Config project’s codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/piotrmurach/tty-config/blob/master/CODE_OF_CONDUCT.md).

## Copyright

Copyright (c) 2018 Piotr Murach. See LICENSE for further details.
