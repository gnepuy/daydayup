[TOC]



## 1. lib 目录结构

```
╰─± tree
.
├── athena
│   ├── api.rb
│   ├── client
│   │   ├── application.rb
│   │   ├── group.rb
│   │   ├── module.rb
│   │   └── pipeline.rb
│   ├── client.rb
│   ├── configuration.rb
│   ├── error.rb
│   ├── objectifiedhash.rb
│   ├── request.rb
│   └── version.rb
└── athena.rb
```



## 2. lib/athena.rb

### 1. 完整代码

```ruby
require 'athena/configuration'
require 'athena/request'
require 'athena/api'
require 'athena/objectifiedhash'
require 'athena/client'
require 'athena/version'

module Athena
  extend Configuration

  # Alias for athena::Client.new
  #
  # @return [Athena::Client]
  def self.client(options = {})
    Athena::Client.new(options)
  end

  # Delegate to Gitlab::Client
  def self.method_missing(method, *args, &block)
    return super unless client.respond_to?(method)
    client.send(method, *args, &block)
  end

  # Delegate to Gitlab::Client
  def self.respond_to_missing?(method_name, include_private = false)
    client.respond_to?(method_name) || super
  end


  def self.http_proxy(address = nil, port = nil, username = nil, password = nil)
    Athena::Request.http_proxy(address, port, username, password)
  end

  # Returns an unsorted array of available client methods.
  #
  # @return [Array<Symbol>]
  def self.actions
    hidden = /endpoint|private_token|auth_token|user_agent|sudo|get|post|put|\Adelete\z|httparty/
    (Athena::Client.instance_methods - Object.methods).reject { |e| e[hidden] }
  end
end

Athena.configure do |config|
  # config.endpoint = 'http://xxx.api.com'
  #
  # Dev Env
  config.endpoint = 'http://xxx.api.com/api/v2'
end
```

### 2. 大致结构

```ruby
# 1、模块定义
module Athena
  ...
end

# 2、默认配置
Athena.configure do |config|
  config.endpoint = 'http://xxx.api.com/api/v2'
end
```

### 3. extend Configuration

拷贝 module **Configuration** 中的所有对象方法

```ruby
module Athena
  extend Configuration
  .....
end
```

### 4. Delegate to Gitlab::Client

```ruby
module Athena
  ......

  # Delegate to Gitlab::Client
  def self.method_missing(method, *args, &block)
    return super unless client.respond_to?(method) # 如果 client 对象, 没有实现 method, 则将消息发给 super
    client.send(method, *args, &block) # 否则将消息转发给 client 对象
  end

  # Delegate to Gitlab::Client
  def self.respond_to_missing?(method_name, include_private = false)
    client.respond_to?(method_name) || super
  end

  ......
end
```

### 5. http_proxy

```ruby
module Athena
  ......

  def self.http_proxy(address = nil, port = nil, username = nil, password = nil)
    Athena::Request.http_proxy(address, port, username, password)
  end

  ......
end
```

### 6. Returns an unsorted array of available client methods

```ruby
module Athena
  ......

  # Returns an unsorted array of available client methods.
  #
  # @return [Array<Symbol>]
  def self.actions
    # 一个正则式, 从对象方法列表中, 过滤掉的一些方法名字
    hidden = /endpoint|private_token|auth_token|user_agent|sudo|get|post|put|\Adelete\z|httparty/

    # 返回 Athena::Client 给外包可见的 对象方法 列表
    (Athena::Client.instance_methods - Object.methods).reject { |e| e[hidden] }
  end

  ......
end
```

### 7. 默认 初始化配置

```ruby
module Athena
  ......

  Athena.configure do |config|
    config.endpoint = 'http://xxx.api.com/api/v2'
  end

......
end
```



## 3. lib/athena/configuration.rb

### 1. 完整代码

```ruby
module Athena
  # global configurations
  module Configuration
    VALID_OPTION_KEYS = %i[endpoint].freeze

    attr_accessor(*VALID_OPTION_KEYS)

    # each time the module is extended, reset
    def self.extened(base)
      base.reset
    end

    # Convenience method to allow configuration options to be set in a block.
    def configure
      yield self
    end

    def reset
      self.endpoint = 'http://xxx.api.com'
    end

    def options
      VALID_OPTION_KEYS.inject({}) do |option, key|
        option.merge!(key => send(key))
      end
    end
  end
end
```

### 2. 大致结构

```ruby
module Athena
  module Configuration
    .....
  end
end
```

### 3. attr_accessor

```ruby
module Athena
  module Configuration
    .......

    VALID_OPTION_KEYS = %i[endpoint].freeze
    attr_accessor(*VALID_OPTION_KEYS)

    .......
  end
end
```

### 4. module is extended callback

```ruby
module Athena
  module Configuration
    .......

    # each time the module is extended, reset
    def self.extened(base)
      base.reset
    end

    .......
  end
end
```

### 5. configure (DSL)

```ruby
module Athena
  module Configuration
    .......

    # Convenience method to allow configuration options to be set in a block.
    def configure
      yield self
    end

    .......
  end
end
```

### 6. reset

```ruby
module Athena
  module Configuration
    .......

    def reset
      self.endpoint = 'http://xxx.api.com'
    end

    .......
  end
end
```

### 7. options

```ruby
module Athena
  module Configuration
    .......

    def options
      VALID_OPTION_KEYS.inject({}) do |option, key|
        option.merge!(key => send(key))
      end
    end

    .......
  end
end
```