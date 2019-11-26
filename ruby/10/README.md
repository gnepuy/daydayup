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
require 'athena/version'
require 'athena/objectifiedhash'
require 'athena/configuration'
require 'athena/error'
require 'athena/request'
require 'athena/api'
require 'athena/client'

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
  extend Configuration # 将 Configuration 对象方法, 全部拷贝到 Athena 模块中
  .....
end
```

### 4. Delegate to Gitlab::Client

```ruby
module Athena
  ......

  # Delegate to Gitlab::Client
  def self.method_missing(method, *args, &block)
    # 如果 client 对象, 没有实现 method, 则将消息发给 super
    return super unless client.respond_to?(method)

    # 否则将消息转发给 client 对象
    # - 1) client: 调用 self.client() 类方法
    # - 2) client.send(method, ...): 调用 Athena::Client 类对象的 方法
    client.send(method, *args, &block)
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
    config.endpoint = 'http://xxx.api.com/api/v2' #=> 1) method_missing => 2) attr_accesssor :endpoint
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

    # 赋值、转发 给 Athena::Client 类对象的的 属性数组
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
      base.reset # 调用 reset 重置设置的 host
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

    # 将所有的属性值, 打包成一个 hash 然后返回
    def options
      VALID_OPTION_KEYS.inject({}) do |option, key|
        option.merge!(key => send(key))
      end
    end

    .......
  end
end
```



## 4、lib/athena/objectifiedhash.rb

### 1. 完整代码

```ruby
module Athena
  # Converts hashes to the objects.
  class ObjectifiedHash
    # Creates a new ObjectifiedHash object.
    def initialize(hash)
      @hash = hash
      @data = hash.each_with_object({}) do |(key, value), data|
        value = ObjectifiedHash.new(value) if value.is_a? Hash
        data[key.to_s] = value
      end
    end

    # @return [Hash] The original hash.
    def to_hash
      @hash
    end
    alias to_h to_hash

    # @return [String] Formatted string with the class name, object id and original hash.
    def inspect
      "#<#{self.class}:#{object_id} {hash: #{@hash.inspect}}"
    end

    # Delegate to ObjectifiedHash.
    def method_missing(key)
      @data.key?(key.to_s) ? @data[key.to_s] : super
    end

    def respond_to_missing?(method_name, include_private = false)
      @hash.keys.map(&:to_sym).include?(method_name.to_sym) || super
    end
  end
end
```

ObjectifiedHash 作用，是将 **hash** 转换成一个 **object** 

### 2. 代码结构

```ruby
module Athena
  class ObjectifiedHash
    ...
  end
end
```

### 3. ObjectifiedHash 使用示例

```ruby
require 'pp'
require 'json'

class ObjectifiedHash
  # Creates a new ObjectifiedHash object.
  def initialize(hash)
    @hash = hash
    @data = hash.each_with_object({}) do |(key, value), data|
      value = ObjectifiedHash.new(value) if value.is_a? Hash
      data[key.to_s] = value
    end
  end

  # @return [Hash] The original hash.
  def to_hash
    @hash
  end
  alias to_h to_hash

  # @return [String] Formatted string with the class name, object id and original hash.
  def inspect
    "#<#{self.class}:#{object_id} {hash: #{@hash.inspect}}"
  end

  # Delegate to ObjectifiedHash.
  def method_missing(key)
    @data.key?(key.to_s) ? @data[key.to_s] : super
  end

  def respond_to_missing?(method_name, include_private = false)
    @hash.keys.map(&:to_sym).include?(method_name.to_sym) || super
  end
end

class Man < ObjectifiedHash
  attr_accessor :name, :favor, :age
end


hash = {
  name: 'xiongzenghui',
  favor: 'hahahahaha',
  age: 99
}

m = ObjectifiedHash.new(hash)
puts m.to_hash
puts m.inspect
puts m.name
puts m.favor
puts m.age
```

```
╰─± ruby main.rb
{:name=>"xiongzenghui", :favor=>"hahahahaha", :age=>99}
#<ObjectifiedHash:70363895612400 {hash: {:name=>"xiongzenghui", :favor=>"hahahahaha", :age=>99}}
xiongzenghui
hahahahaha
99
```

### 4. 类似 JSON.parse(.., object_class: OpenStruct)

```ruby
obj = JSON.parse(File.read('xx.json'), object_class: OpenStruct)
puts pbj.name
puts pbj.favor
puts pbj.age
```



## 5、lib/athena/error.rb

### 1. 完整代码

```ruby
module Gitlab
  module Error
    # Custom error class for rescuing from all Gitlab errors.
    class Error < StandardError; end

    # Raised when API endpoint credentials not configured.
    class MissingCredentials < Error; end

    # Raised when impossible to parse response body.
    class Parsing < Error; end

    # Custom error class for rescuing from HTTP response errors.
    class ResponseError < Error
      POSSIBLE_MESSAGE_KEYS = %i[message error_description error].freeze

      def initialize(response)
        @response = response
        super(build_error_message)
      end

      # Status code returned in the HTTP response.
      #
      # @return [Integer]
      def response_status
        @response.code
      end

      # Body content returned in the HTTP response
      #
      # @return [String]
      def response_message
        @response.parsed_response.message
      end

      private

      # Human friendly message.
      #
      # @return [String]
      def build_error_message
        parsed_response = classified_response
        message = check_error_keys(parsed_response)
        "Server responded with code #{@response.code}, message: " \
        "#{handle_message(message)}. " \
        "Request URI: #{@response.request.base_uri}#{@response.request.path}"
      end

      # Error keys vary across the API, find the first key that the parsed_response
      # object responds to and return that, otherwise return the original.
      def check_error_keys(resp)
        key = POSSIBLE_MESSAGE_KEYS.find { |k| resp.respond_to?(k) }
        key ? resp.send(key) : resp
      end

      # Parse the body based on the classification of the body content type
      #
      # @return parsed response
      def classified_response
        if @response.respond_to?('headers')
          @response.headers['content-type'] == 'text/plain' ? { message: @response.to_s } : @response.parsed_response
        else
          @response.parsed_response
        end
      end

      # Handle error response message in case of nested hashes
      def handle_message(message)
        case message
        when Gitlab::ObjectifiedHash
          message.to_h.sort.map do |key, val|
            "'#{key}' #{(val.is_a?(Hash) ? val.sort.map { |k, v| "(#{k}: #{v.join(' ')})" } : [val].flatten).join(' ')}"
          end.join(', ')
        when Array
          message.join(' ')
        else
          message
        end
      end
    end

    # Raised when API endpoint returns the HTTP status code 400.
    class BadRequest < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 401.
    class Unauthorized < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 403.
    class Forbidden < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 404.
    class NotFound < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 405.
    class MethodNotAllowed < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 406.
    class NotAcceptable < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 409.
    class Conflict < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 422.
    class Unprocessable < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 429.
    class TooManyRequests < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 500.
    class InternalServerError < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 502.
    class BadGateway < ResponseError; end

    # Raised when API endpoint returns the HTTP status code 503.
    class ServiceUnavailable < ResponseError; end

    # HTTP status codes mapped to error classes.
    STATUS_MAPPINGS = {
      400 => BadRequest,
      401 => Unauthorized,
      403 => Forbidden,
      404 => NotFound,
      405 => MethodNotAllowed,
      406 => NotAcceptable,
      409 => Conflict,
      422 => Unprocessable,
      429 => TooManyRequests,
      500 => InternalServerError,
      502 => BadGateway,
      503 => ServiceUnavailable
    }.freeze
  end
end
```

### 2. 代码结构

```ruby
module Gitlab
  module Error
    ...
  end
end
```

### 3. Error

```ruby
class Error < StandardError; end
```

### 4. MissingCredentials

```ruby
class MissingCredentials < Error; end
```

### 5. Parsing

```ruby
class Parsing < Error; end
```

### 6. ResponseError

```ruby
class ResponseError < Error
  POSSIBLE_MESSAGE_KEYS = %i[message error_description error].freeze

  def initialize(response)
    @response = response
    super(build_error_message)
  end

  # Status code returned in the HTTP response.
  #
  # @return [Integer]
  def response_status
    @response.code
  end

  # Body content returned in the HTTP response
  #
  # @return [String]
  def response_message
    @response.parsed_response.message
  end

  private

  # Human friendly message.
  #
  # @return [String]
  def build_error_message
    parsed_response = classified_response
    message = check_error_keys(parsed_response)
    "Server responded with code #{@response.code}, message: " \
    "#{handle_message(message)}. " \
    "Request URI: #{@response.request.base_uri}#{@response.request.path}"
  end

  # Error keys vary across the API, find the first key that the parsed_response
  # object responds to and return that, otherwise return the original.
  def check_error_keys(resp)
    key = POSSIBLE_MESSAGE_KEYS.find { |k| resp.respond_to?(k) }
    key ? resp.send(key) : resp
  end

  # Parse the body based on the classification of the body content type
  #
  # @return parsed response
  def classified_response
    if @response.respond_to?('headers')
      @response.headers['content-type'] == 'text/plain' ? { message: @response.to_s } : @response.parsed_response
    else
      @response.parsed_response
    end
  end

  # Handle error response message in case of nested hashes
  def handle_message(message)
    case message
    when Athena::ObjectifiedHash
      message.to_h.sort.map do |key, val|
        "'#{key}' #{(val.is_a?(Hash) ? val.sort.map { |k, v| "(#{k}: #{v.join(' ')})" } : [val].flatten).join(' ')}"
      end.join(', ')
    when Array
      message.join(' ')
    else
      message
    end
  end
end
```

### 7. 其他继承自 ResponseError 的子类

```ruby
# Raised when API endpoint returns the HTTP status code 400.
class BadRequest < ResponseError; end

# Raised when API endpoint returns the HTTP status code 401.
class Unauthorized < ResponseError; end

# Raised when API endpoint returns the HTTP status code 403.
class Forbidden < ResponseError; end

# Raised when API endpoint returns the HTTP status code 404.
class NotFound < ResponseError; end

# Raised when API endpoint returns the HTTP status code 405.
class MethodNotAllowed < ResponseError; end

# Raised when API endpoint returns the HTTP status code 409.
class Conflict < ResponseError; end

# Raised when API endpoint returns the HTTP status code 422.
class Unprocessable < ResponseError; end

# Raised when API endpoint returns the HTTP status code 429.
class TooManyRequests < ResponseError; end

# Raised when API endpoint returns the HTTP status code 500.
class InternalServerError < ResponseError; end

# Raised when API endpoint returns the HTTP status code 502.
class BadGateway < ResponseError; end

# Raised when API endpoint returns the HTTP status code 503.
class ServiceUnavailable < ResponseError; end
```

### 8. HTTP status codes mapped to error classes.

```ruby
STATUS_MAPPINGS = {
  400 => BadRequest,
  401 => Unauthorized,
  403 => Forbidden,
  404 => NotFound,
  405 => MethodNotAllowed,
  406 => NotAcceptable,
  409 => Conflict,
  422 => Unprocessable,
  429 => TooManyRequests,
  500 => InternalServerError,
  502 => BadGateway,
  503 => ServiceUnavailable
}.freeze
```



## 6、lib/athena/request.rb

### 1. 完整代码

```ruby
require 'httparty'
require 'json'

module Gitlab
  # @private
  class Request
    include HTTParty
    format :json
    headers 'Accept' => 'application/json', 'Content-Type' => 'application/x-www-form-urlencoded'
    parser(proc { |body, _| parse(body) })

    attr_accessor :private_token, :endpoint

    # Converts the response body to an ObjectifiedHash.
    def self.parse(body)
      body = decode(body)

      if body.is_a? Hash
        ObjectifiedHash.new body
      elsif body.is_a? Array
        PaginatedResponse.new(body.collect! { |e| ObjectifiedHash.new(e) })
      elsif body
        true
      elsif !body
        false
      elsif body.nil?
        false
      else
        raise Error::Parsing, "Couldn't parse a response body"
      end
    end

    # Decodes a JSON response into Ruby object.
    def self.decode(response)
      response ? JSON.load(response) : {}
    rescue JSON::ParserError
      raise Error::Parsing, 'The response is not a valid JSON'
    end

    %w[get post put delete].each do |method|
      define_method method do |path, options = {}|
        httparty_config(options)

        unless options[:unauthenticated]
          options[:headers] ||= {}
          options[:headers].merge!(authorization_header)
        end

        validate self.class.send(method, @endpoint + path, options)
      end
    end

    # Checks the response code for common errors.
    # Returns parsed response for successful requests.
    def validate(response)
      error_klass = Error::STATUS_MAPPINGS[response.code]
      raise error_klass, response if error_klass

      parsed = response.parsed_response
      parsed.client = self if parsed.respond_to?(:client=)
      parsed.parse_headers!(response.headers) if parsed.respond_to?(:parse_headers!)
      parsed
    end

    # Sets a base_uri and default_params for requests.
    # @raise [Error::MissingCredentials] if endpoint not set.
    def request_defaults(sudo = nil)
      raise Error::MissingCredentials, 'Please set an endpoint to API' unless @endpoint

      self.class.default_params sudo: sudo
      self.class.default_params.delete(:sudo) if sudo.nil?
    end

    private

    # Returns an Authorization header hash
    #
    # @raise [Error::MissingCredentials] if private_token and auth_token are not set.
    def authorization_header
      raise Error::MissingCredentials, 'Please provide a private_token or auth_token for user' unless @private_token

      if @private_token.size < 21
        { 'PRIVATE-TOKEN' => @private_token }
      else
        { 'Authorization' => "Bearer #{@private_token}" }
      end
    end

    # Set HTTParty configuration
    # @see https://github.com/jnunemaker/httparty
    def httparty_config(options)
      options.merge!(httparty) if httparty
    end
  end
end
```

### 2. 代码结构

```ruby
module Athena
  class Request
    ....
  end
end
```

### 3. 包含 HTTParty 模块、初始化

```ruby
module Athena
  class Request
    include HTTParty
    format :json
    headers 'Accept' => 'application/json', 
    				'Content-Type' => 'application/x-www-form-urlencoded'
    parser(proc {|body, _| parse(body)})
		
		attr_accessor :endpoint
		.....
  end
end
```

### 4. [httparty 使用](https://github.com/jnunemaker/httparty)

#### 1. Use the class methods to get down to business quickly

```ruby
response = HTTParty.get('http://api.stackexchange.com/2.2/questions?site=stackoverflow')
puts response.body, response.code, response.message, response.headers.inspect
```

#### 2. wrap things up in your own class

##### 1. Class

```ruby
class StackExchange
  include HTTParty
  base_uri 'api.stackexchange.com'

  def initialize(service, page)
    @options = { query: { site: service, page: page } }
  end

  def questions
    self.class.get("/2.2/questions", @options)
  end

  def users
    self.class.get("/2.2/users", @options)
  end
end
```

##### 2. Instance

```ruby
stack_exchange = StackExchange.new("stackoverflow", 1)
puts stack_exchange.questions
puts stack_exchange.users
```

### 5. parse

```ruby
# Converts the response body to an ObjectifiedHash.
def self.parse(body)
  body = decode(body)

  if body.is_a? Hash
    ObjectifiedHash.new body
  elsif body.is_a? Array
    # 这句如果你没有【分页】需求，可以注释掉
    # PaginatedResponse.new(body.collect! { |e| ObjectifiedHash.new(e) })
  elsif body
    true
  elsif !body
    false
  elsif body.nil?
    false
  else
    raise Error::Parsing, "Couldn't parse a response body"
  end
end
```

### 6. decode

```ruby
# Decodes a JSON response into Ruby object.
def self.decode(response)
  response ? JSON.load(response) : {}
rescue JSON::ParserError
  raise Error::Parsing, 'The response is not a valid JSON'
end

%w[get post put delete].each do |method|
  define_method method do |path, options = {}|
    httparty_config(options)

    unless options[:unauthenticated]
      options[:headers] ||= {}
      options[:headers].merge!(authorization_header)
    end

    validate self.class.send(method, @endpoint + path, options) #=> validate()
  end
end
```

### 7. validate

```ruby
# Checks the response code for common errors.
# Returns parsed response for successful requests.
def validate(response)
  error_klass = Error::STATUS_MAPPINGS[response.code]
  raise error_klass, response if error_klass

  parsed = response.parsed_response
  parsed.client = self if parsed.respond_to?(:client=)
  parsed.parse_headers!(response.headers) if parsed.respond_to?(:parse_headers!)
  parsed
end
```

展开 `Error::STATUS_MAPPINGS` 为

```ruby
def validate(response)
  error_klass = case response.code
                when 400 then
                  Error::BadRequest
                when 401 then
                  Error::Unauthorized
                when 403 then
                  Error::Forbidden
                when 404 then
                  Error::NotFound
                when 405 then
                  Error::MethodNotAllowed
                when 409 then
                  Error::Conflict
                when 422 then
                  Error::Unprocessable
                when 429 then
                  Error::TooManyRequests
                when 500 then
                  Error::InternalServerError
                when 502 then
                  Error::BadGateway
                when 503 then
                  Error::ServiceUnavailable
                else
                  nil
                end
  raise error_klass, response if error_klass

  parsed = response.parsed_response
  parsed.client = self if parsed.respond_to?(:client=)
  parsed
end
```

### 8. request_defaults

```ruby
# Sets a base_uri and default_params for requests.
# @raise [Error::MissingCredentials] if endpoint not set.
def request_defaults(sudo = nil)
  raise Error::MissingCredentials, 'Please set an endpoint to API' unless @endpoint

  self.class.default_params sudo: sudo
  self.class.default_params.delete(:sudo) if sudo.nil?
end
```



## 7、lib/athena/api.rb

```ruby
# frozen_string_literal: true
module Athena
  # @private
  class API < Request
    # @private
    attr_accessor(*Configuration::VALID_OPTION_KEYS)

    # Creates a new API.
    # @raise [Error:MissingCredentials]
    #
    def initialize(options = {})
      options = Athena.options.merge(options)
      (Configuration::VALID_OPTION_KEYS + [:auth_token]).each do |key|
        send("#{key}=", options[key]) if options[key]
      end
      request_defaults
    end
  end
end
```

- API 继承自 Request



## 8、lib/athena/client.rb

### 1. 完整代码

```ruby
module Athena
  class Client < API
    # 1、require 导入 client/ 下面所有的 xx.rb 
    # 相当于有如下几个 require
    # -1) require_relative 'client/application'
    # -2) require_relative 'client/group'
    # -3) require_relative 'client/module'
    # -4) require_relative 'client/pipeline'
    Dir[File.expand_path('client/*.rb', __dir__)].each { |f| require f }
		
    # 2、包含 其他类型 API 接口的 module,
    # 这样就能直接使用 Athena::Client.xx 调用
    # Pipeline、Application、Groups、Module
    # 这几个 module 中定义的【对象方法】
    include Pipeline		#=> client/pipeline.rb
    include Application	#=> client/application.rb
    include Groups			#=> client/group.rb
    include Module			#=> client/module.rb

    # Text representation of the client, masking private token.
    #
    # @return [String]
    def inspect
      # do nothing.
    end
    
    def url_encode(url)
      URI.encode(url.to_s, /\W/)
    end
  end
end
```

- Client 继承自 API

### 2. 代码结构

```ruby
module Athena
  class Client < API
    ....
  end
end
```

- Client 使用 **Class** 修饰为一个 **类**
- 因为 **模块** 中的 **类方法** 不能直接被外部调用

### 3. require 导入 `client/` 下面所有的 xx.rb 

```ruby
module Athena
  class Client < API
    # 1、require 导入 client/ 下面所有的 xx.rb 
    # 相当于有如下几个 require
    # -1) require_relative 'client/application'
    # -2) require_relative 'client/group'
    # -3) require_relative 'client/module'
    # -4) require_relative 'client/pipeline'
    Dir[File.expand_path('client/*.rb', __dir__)].each { |f| require f }
    
    ........
  end
end
```

### 4. 包含 其他类型 API 接口的 module

```ruby
module Athena
  class Client < API
    ......
      
    # 2、包含 其他类型 API 接口的 module,
    # 这样就能直接使用 Athena::Client.xx 调用
    # Pipeline、Application、Groups、Module
    # 这几个 module 中定义的【对象方法】
    include Pipeline		#=> client/pipeline.rb
    include Application	#=> client/application.rb
    include Groups			#=> client/group.rb
    include Module			#=> client/module.rb
    
    ......
  end
end
```





## 9、`client/*.rb ` ==扩展== 其他类型 API

### 1. 目录结构

```
╰─± tree lib/athena/client
lib/athena/client
├── application.rb
├── group.rb
├── module.rb
└── pipeline.rb
```

### 2. Gitlab::Client

- Gitlab 是一个 **module**
- Client 是一个 **class**

```ruby
module Gitlab
  class Client < API
    ...
  end
end
```

### 3. 两种 ==扩展模块== 写法

#### 1. Gitlab --> Client --> 扩展模块

```ruby
module Gitlab
  class Client
  end
end

module Gitlab
  class Client
    module App
      def run; puts '🚗 app ...'; end
    end
  end
end

module Gitlab
  class Client
    module Group
      def run; puts '🚙 group ...'; end
    end
  end
end

module Gitlab
  class Client
    include App
    include Group
  end
end
```

#### 2. class Gitlab::Client --> 扩展模块

```ruby
module Gitlab
  class Client
  end
end

class Gitlab::Client
  module App
    def run; puts '🚗 app ...'; end
  end
end

class Gitlab::Client
  module Group
    def run; puts '🚙 group ...'; end
  end
end

module Gitlab
  class Client
    include App
    include Group
  end
end
```

#### 3. 对比2种 ==扩展模块== 的写法

```ruby
module Gitlab		# L1
  class Client	# L2
    module App	# L3
      def run; puts '🚗 app ...'; end
    end
  end
end
```

```ruby
class Gitlab::Client 	#=> L1 + L2
  module App					#=> L3
    def run; puts '🚗 app ...'; end
  end
end
```

### 4. client/application.rb

```ruby
class Athena::Client
  module Application
    def apps(options = {})
      # 调用 HttpParty 发网络请求
      get('/apps', query: options)
    end
  end
end
```

### 5. client/module.rb

```ruby
class Athena::Client
  module Module
    def module_info(module_name)
      # 调用 HttpParty 发网络请求
      get('/modules_info', query: { 'module_name' => module_name })
    end
  end
end
```

### 6. 扩展其他类型的 API

```ruby
class Athena::Client
  module 扩展模块名
    def 扩展方法名(参数)
      # 调用 HttpParty 发网络请求
      get('/path', query: { '参数' => '值' })
    end
  end
end
```



## 10、调用 API

### 1. Application (client/application.rb)

```ruby
apps = Athena.apps
puts apps
```

### 2. Module (client/module.rb)

```ruby
module_info = Athena.module_info('demo3')
pp module_info.data	#=> 自动将 response json 转换为 object
pp module_info.data.pipelines #=> 同上
pp module_info.data.members	#=> 同上
```

