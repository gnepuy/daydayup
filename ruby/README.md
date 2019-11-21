[TOC]

## 觉得有用给点吧 ~

<img src="dashang.jpg" width ="250" height ="250"/>



## 1. 😀 Ruby 这门语言

[点击我](01/01.md)



## 2. 😁 Ruby 学习中即将接触到的各种概念

[点击我](02/01.md)



## 3. 😂 使用 RVM 管理 Ruby 开发环境

[点击我](03/01.md)



## 4. 😨 令初学者疑惑的 require: cannot load such file -- xxx (LoadError)

[点击我](04/01.md)



## 5. 🤣 Bundler : 管理 gem 依赖

- [01 - 一个 Ruby 软件包](05/01/README.md)
- [02 - Bundler 就是 这些 Ruby 软件的 包管理器](05/02/README.md)
- [03 - Gemfile 语法](05/03/README.md)
- [04 - bundle install](05/04/README.md)
- [05 - bundle update](05/05/README.md)
- [06 - bundle exec](05/06/README.md)
- [07 - bundler Shelling Out](05/07/README.md)
- [08 - 其他](05/08/README.md)



## 6. 😇 Ruby 基础语法

### 1. 常用数据类型

- [01 - String](06/01/01.md)
- [02 - Bool](06/01/02.md)
- [03 - nil](06/01/03.md)
- [04 - Symbol](06/01/04.md)
- [05 - Array](06/01/05.md)
- [06 - Hash](06/01/06.md)
- [07 - Struct](06/01/07.md)

### 2. 变量

[点击我](06/02/01.md)

### 3. 常用运算符

- [01 - 单引号 vs 双引号](06/03/01.md)

- [02 - % 运算符](06/03/02.md)

### 4. 流程控制

if/else/switch/times/break/continue ….

### 5. 方法

- [01 - 全局 (global) 方法](06/05/01.md)
- [02 - 对象 (instance) 方法](06/05/02.md)
- [03 - 类 (class) 方法](06/05/03.md)
- [04 - 单例类 (singleton class) 方法](06/05/04.md)
- [05 - 方法参数](06/05/05.md)
- [06 - 获取方法](06/05/06.md)

### 6. 代码块

[点击我](06/06/01.md)

### 7. 方法 与 代码块

[点击我](06/07/01.md)

### 8. 类与对象

- [01 - 面向对象基础](06/08/01.md)
- [02 - 对象的拷贝](06/08/02.md)
- [03 - 对象比较](06/08/03.md)
- [04 - 继承](06/08/04.md)
- [05 - 异常](06/08/05.md)



## 7.  😄 Ruby 高级语法

### 1. 模块 (module)

- [01 - 基础使用](07/01/01.md)
- [02- include、extend、prepend](07/01/02.md)
- [03 - Gitlab Ruby API 封装](07/01/03.md)
- [04 - require、require_relative、load、autoload](07/01/04.md)

### 2. 消息


- [01 - message send](07/02/01.md)

- [02 - method missing](07/02/02.md)

- [03 - message forward](07/02/03.md)

### 3. DSL

[点击我](07/03.md)

### 4. 各种钩子 (hook)

[点击我](07/04.md)

### 5. patch

[点击我](07/05.md)



## 8. 😅 Ruby 设计模式

### 1. SOLID 原则

[点击我](08/01.md)

### 2. 设计模式

- [01 - 单例](08/02/01.md)
- [02 - 代理](08/02/02.md)





## 9. 😉 手把手教你开发并上线一个 Ruby 软件

- 本地的 开发、调试

- 发布到 rubygems.org

- 本地 调试 Ruby 开源库



## 10. ⚽️ CocoaPods

### 1. cocoapods 工作原理



### 2. cocoapods.gemspec

```ruby
# encoding: UTF-8
require File.expand_path('../lib/cocoapods/gem_version', __FILE__)
require 'date'

Gem::Specification.new do |s|
  .......................................................

  s.files = Dir["lib/**/*.rb"] + %w{ bin/pod bin/sandbox-pod README.md LICENSE CHANGELOG.md }

  s.executables   = %w{ pod sandbox-pod }
  s.require_paths = %w{ lib }

  # Link with the version of CocoaPods-Core
  s.add_runtime_dependency 'cocoapods-core',        "= #{Pod::VERSION}"

  s.add_runtime_dependency 'claide',                '>= 1.0.2', '< 2.0'
  s.add_runtime_dependency 'cocoapods-deintegrate', '>= 1.0.3', '< 2.0'
  s.add_runtime_dependency 'cocoapods-downloader',  '>= 1.2.2', '< 2.0'
  s.add_runtime_dependency 'cocoapods-plugins',     '>= 1.0.0', '< 2.0'
  s.add_runtime_dependency 'cocoapods-search',      '>= 1.0.0', '< 2.0'
  s.add_runtime_dependency 'cocoapods-stats',       '>= 1.0.0', '< 2.0'
  s.add_runtime_dependency 'cocoapods-trunk',       '>= 1.4.0', '< 2.0'
  s.add_runtime_dependency 'cocoapods-try',         '>= 1.1.0', '< 2.0'
  s.add_runtime_dependency 'molinillo',             '~> 0.6.6'
  s.add_runtime_dependency 'xcodeproj',             '>= 1.11.1', '< 2.0'

  ## Version 5 needs Ruby 2.2, so we specify an upper bound to stay compatible with system ruby
  s.add_runtime_dependency 'activesupport', '>= 4.0.2', '< 5'
  s.add_runtime_dependency 'colored2',       '~> 3.1'
  s.add_runtime_dependency 'escape',        '~> 0.0.4'
  s.add_runtime_dependency 'fourflusher',   '>= 2.3.0', '< 3.0'
  s.add_runtime_dependency 'gh_inspector',  '~> 1.0'
  s.add_runtime_dependency 'nap',           '~> 1.0'
  s.add_runtime_dependency 'ruby-macho',    '~> 1.4'

  s.add_development_dependency 'bacon', '~> 1.1'
  s.add_development_dependency 'bundler', '~> 1.3'
  s.add_development_dependency 'rake', '~> 10.0'

  .......................................................
end
```

cocoapods 主要由如下几个 gem 组成

- 1) cocoapods-core
- 2) claide
- 3) cocoapods-deintegrate
- 4) cocoapods-downloader
- 5) cocoapods-plugins
- 6) cocoapods-search
- 7) cocoapods-stats
- 8) cocoapods-trunk
- 9) cocoapods-try
- 10) molinillo
- 11) xcodeproj

### 3. Podfile

cocoapods_podfile

### 4. podspec

cocoapods_podspec

### 5. xcodeproj



### 6. cocoapods hooks



### 7. cocoapods-plugins



### 8. cocoapods wrapper





## 11.  🏀 fastlane



