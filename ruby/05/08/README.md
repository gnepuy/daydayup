[TOC]



## 1.  use Bundler in a single-file Ruby script

- 1) To use Bundler in a single-file script, add require **'bundler/inline'** at the top of your Ruby file.
- 2) use the **gemfile method** to **declare any gem** sources and gems that you need

```ruby
##################################################################################
# bundler_inline_example.rb
##################################################################################
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'json', require: false
  gem 'nap', require: 'rest'
  gem 'cocoapods', '~> 0.34.1'
end

puts 'Gems installed and loaded!'
puts "The nap gem is at version #{REST::VERSION}"
```



## 2. bundler plugin hook bundle process

https://bundler.io/v2.0/guides/bundler_plugins.html

### 1. Gemfile 中指定安装 plugin

```ruby
plugin 'my_plugin' # Installs from Rubygems
plugin 'my_plugin', path: '/path/to/plugin' # Installs from a path
plugin 'my_plugin', git: 'https://github.com:repo/my_plugin.git' # Installs from Git
```

### 2. 创建一个自己的 plugin

https://bundler.io/v2.0/guides/bundler_plugins.html#getting-started-with-development

比如实现 **拦截 bundle install** 的过程:

```ruby
Bundler::Plugin.add_hook('before-install-all') do |dependencies|
  # Do something with the dependencies
end
```



## 3. can't find gem bundler with executable bundle

### 1. install ruby gem 报错如下

```ruby
Traceback (most recent call last):
	4: from /Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/ruby_executable_hooks:24:in `<main>'
	3: from /Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/ruby_executable_hooks:24:in `eval'
	2: from /Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/bundle:23:in `<main>'
	1: from /Users/Fuhanyu/.rvm/rubies/ruby-2.5.1/lib/ruby/2.5.0/rubygems.rb:308:in `activate_bin_path'
/Users/Fuhanyu/.rvm/rubies/ruby-2.5.1/lib/ruby/2.5.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
Traceback (most recent call last):
	4: from /Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/ruby_executable_hooks:24:in `<main>'
	3: from /Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/ruby_executable_hooks:24:in `eval'
	2: from /Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/bundle:23:in `<main>'
	1: from /Users/Fuhanyu/.rvm/rubies/ruby-2.5.1/lib/ruby/2.5.0/rubygems.rb:308:in `activate_bin_path'
/Users/Fuhanyu/.rvm/rubies/ruby-2.5.1/lib/ruby/2.5.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
rake aborted!
LoadError: cannot load such file -- bundler/gem_tasks
/private/tmp/kanshan/Rakefile:1:in `<top (required)>'
/Users/Fuhanyu/.rvm/gems/ruby-2.5.1/gems/rake-12.3.1/exe/rake:27:in `<top (required)>'
/Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/ruby_executable_hooks:24:in `eval'
/Users/Fuhanyu/.rvm/gems/ruby-2.5.1/bin/ruby_executable_hooks:24:in `<main>'
(See full trace by running task with --trace)
```

### 2. 关键报错

```ruby
can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
rake aborted!
```

看起来好像是与 **bundler 版本** 相关的问题.

### 3. 报错的机器上的 bundler 版本: 2.0.2

```
 ~  bundle -v
Bundler version 2.0.2
```

### 4. 生成 ruby gem 使用的 bundler 版本: 2.0.1

```ruby
# Gemfile.lock

.......

DEPENDENCIES
  bundler (~> 2.0.1)
  kanshan!
  rake (~> 10.0)
  rspec (~> 3.0)
  rubocop (= 0.59.0)
  simplecov
  simplecov-html

BUNDLED WITH
   2.0.1
```

发现这两个是不一致的.

### 5. 尝试让 本机 保持与 ruby-gem 一样的 bundler 版本 2.0.1

```
gem uninstall bundler
gem install bundler -v 2.0.1
```

再次安装 ruby-gem 就 ok 了..

