[TOC]



## 1. gem install xxx

```
gem install <软件的名字>
```

这样方式的，一次只能安装 **1个** 软件包。

那有方式一次性安装 **多个** 软件包吗？



## 2.  Gemfile + bundle install 批量安装 ruby 软件包

### 1. Gemfile 指定所有安装的 ruby 包名

```ruby
source "https://rubygems.org"
gem 'colored'
gem 'gitlab'
```

如果非要是有 **方法调用** 的形式：

```ruby
source("https://rubygems.org")
gem('colored')
gem('gitlab')
```

### 2. bundle install

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/Wocaonima on master✘✘✘ using ‹ruby-2.6.0›
╰─± bundle install
Using bundler 2.0.2
Using colored 1.2
Using mime-types-data 3.2019.0331
Using mime-types 3.2.2
Using multi_xml 0.6.0
Using httparty 0.17.0
Using unicode-display_width 1.6.0
Using terminal-table 1.8.0
Using gitlab 4.11.0
Bundle complete! 2 Gemfile dependencies, 9 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

一次性安装了 colored 和 gitlab 两个 ruby 包。

### 3. 会在 bundle install 目录下生成 Gemfile.lock 文件

> Gemfile.lock 文件的作用, ==锁死== gem 版本号

```
╰─± cat Gemfile.lock
GEM
  remote: https://rubygems.org/
  specs:
    colored (1.2)
    gitlab (4.11.0)
      httparty (~> 0.14, >= 0.14.0)
      terminal-table (~> 1.5, >= 1.5.1)
    httparty (0.17.0)
      mime-types (~> 3.0)
      multi_xml (>= 0.5.2)
    mime-types (3.2.2)
      mime-types-data (~> 3.2015)
    mime-types-data (3.2019.0331)
    multi_xml (0.6.0)
    terminal-table (1.8.0)
      unicode-display_width (~> 1.1, >= 1.1.1)
    unicode-display_width (1.6.0)

PLATFORMS
  ruby

DEPENDENCIES
  colored
  gitlab

BUNDLED WITH
   2.0.2
```

- 根节点：
  - GEM
  - PLATFORMS
  - DEPENDENCIES
  - BUNDLED WITH
- GEM 节点：记录了所有安装的 gem 软件包
- PLATFORMS 节点：ruby
- DEPENDENCIES 节点：当前项目依赖的 gem 软件包（Gemfile显示指定的）
- BUNDLED WITH 节点：当前 `bundler -v` 版本号



