[TOC]



## 1. source

### 1. ==global== source

```ruby
source "https://your_ruby_gem_server.url"

gem 'xxx'
gem 'yyy'
```

都是从 **https://your_ruby_gem_server.url** 下载 gem 包.

### 2. source ==block==

```ruby
source "https://your_ruby_gem_server.url" do
  gem 'xxx'
end

gem 'yyy'
```

- 1) **xxx** 从 **https://your_ruby_gem_server.url** 下载
- 2) **yyy** 从 **默认** 的 **https://rubygems.org** 下载

### 3. gem xxx, source '...'

```ruby
gem 'xxx', source: "https://your_ruby_gem_server.url"
```

单独指定某一个 gem 包的下载源



## 2. version

### 1. 主流的集中引用 gem 包的写法

```ruby
gem 'nokogiri'              # 1、optimistic
gem 'nokogiri', '>=1.4.2'   # 2、optimistic
gem 'nokogiri', '~>1.4.2'   # 3、pessimistic
gem 'nokogiri', '~>1.4'     # 4、pessimistic
gem 'nokogiri', '1.4.2'     # 5、super pessimistic
```

代表的含义：

```ruby
gem 'nokogiri'              # 1、【任何】版本
gem 'nokogiri', '>=1.4.2'   # 2、【任何】+【>= 1.4.2】的版本
gem 'nokogiri', '~>1.4.2'   # 3、【>= 1.4.2】 && 【< 1.5.0】的版本
gem 'nokogiri', '~>1.4'     # 4、【>= 1.4.0】 && 【< 2.0.0】的版本
gem 'nokogiri', '1.4.2'     # 5、只能【== 1.4.2】的版本
```

### 2. 版本号 (major.minor.patch) 兼容性

nokogiri **1.2.3** 为例：

- 1) 1 → Major 版本, 在 **接口重构** 情况下 Major Version 会增加，API **不一定向后兼容**
- 2) 2 → Minor 版本, 在 **增加新特性** 情况下 Minor Version 会增加，并且 API **保持向后兼容**
- 3) 3 → Patch 版本, 在 **bug fix** 的情况下 Patch Version 会增加，并且 API **保持向后兼容**

### 3. bad 版本引用

- 1) **第1种** 方式 **很少人** 采用, 因为一旦 **升级很容易**, 可能因为 **API 不兼容** 导致你的项目爆掉

- 2) **第5种** 指定版本号
  - 1) 需要知道 **最新版本** 具体是多少
  - 2) 并且一个一个 修改 版本号, 增加了升级的复杂度
  - 3) 而实际上 **锁定版本号** 的项目几乎没人去升级

### 4. good 版本引用

- 1) **第3种** 方案（super pessimistic）也可以, 但不是最好的方式
  - 通常是想 **锁定版本号** 求稳定
  - 只会自动更新 **path** , 即只会找到 **bug fix** 最新版 (直接运行 **bundle update 自动修改** 升级 gem)
  - 但 **不会** 自动更新 **minor** , 就 **不会** 找到 **新特性** 最新版
- 2) **第2种** 最好的
  - 既 **不会出现 API 不兼容** 问题
  - 又会 **及时升级** 到 **没有 bug** 的版本

### 5. cocoapods 中 gem 版本依赖写法

写法格式:

```
gem 'xxx', >= a.b.c', '< (a+1).0'
```

源码：

```ruby
s.add_runtime_dependency 'claide',                '>= 1.0.2', '< 2.0'
s.add_runtime_dependency 'cocoapods-deintegrate', '>= 1.0.3', '< 2.0'
s.add_runtime_dependency 'cocoapods-downloader',  '>= 1.2.2', '< 2.0'
s.add_runtime_dependency 'cocoapods-plugins',     '>= 1.0.0', '< 2.0'
s.add_runtime_dependency 'cocoapods-search',      '>= 1.0.0', '< 2.0'
s.add_runtime_dependency 'cocoapods-stats',       '>= 1.0.0', '< 2.0'
s.add_runtime_dependency 'cocoapods-trunk',       '>= 1.3.1', '< 2.0'
s.add_runtime_dependency 'cocoapods-try',         '>= 1.1.0', '< 2.0'
s.add_runtime_dependency 'molinillo',             '~> 0.6.6'
s.add_runtime_dependency 'xcodeproj',             '>= 1.10.0', '< 2.0'
```

### 6. gem bump

**gem bump** command which will bump the gem version to the **next patch** level

```
$ gem bump --version minor # bumps to the next minor version
$ gem bump --version major # bumps to the next major version
$ gem bump --version 1.1.1 # bumps to the specified version
```



## 3. dependency

### 1. git、tag、branch、ref

```ruby
gem 'nokogiri', '1.7.0.1', git: 'https://github.com/sparklemotion/nokogiri', branch: 'master'
```

```ruby
gem 'nokogiri', git: 'https://github.com/rack/rack.git', tag: '2.0.1'
```

```ruby
gem 'nokogiri', git: 'https://github.com/rack/rack.git', branch: 'rack-1.5'
```

```ruby
gem 'nokogiri', git: 'https://github.com/rack/rack.git', ref: '0bd839d'
```

### 2. path (适合本地调试)

```ruby
gem 'nokogiri', path: '/path/to/'
```

### 3. location gemspec

use the **glob** option to specify the **location** of its **xx.gemspec**

```ruby
gem 'cf-copilot', git: 'https://github.com/cloudfoundry/copilot', glob: 'sdk/ruby/*.gemspec'
```

### 4. git repository containing multiple xx.gemspec

```ruby
git 'https://github.com/xxx.git' do
  gem 'xxx' #=> xxx.gemspec
  gem 'yyy' #=> yyy.gemspec
  gem 'zzz' #=> zzz.gemspec
end
```

### 5. HTTP(S), SSH, or git

```ruby
gem 'rack', git: 'https://github.com/rack/rack.git'
gem 'rack', git: 'git@github.com:rack/rack.git'
gem 'rack', git: 'git://github.com/rack/rack.git'
```

### 6. Custom git sources

```ruby
git_source(:stash){ |repo_name| "https://stash.corp.acme.pl/#{repo_name}.git" }
gem 'rails', stash: 'forks/rails'
```

### 7. xxx. gem 'xxx', ==require: false==

….



## 4. ==group== 隔离 不同的 gem 依赖

### 1. Gemfile

```ruby
# These gems are in the :default group
gem 'nokogiri'
gem 'sinatra'

gem 'wirble', group: :development

group :test do
  gem 'faker'
  gem 'rspec'
end

group :test, :development do
  gem 'capybara'
  gem 'rspec-rails'
end

gem 'cucumber', group: [:cucumber, :test]
```

### 2. 排除 test 和 development 这两个组内的 gem 

```
bundle install --without test development
```

