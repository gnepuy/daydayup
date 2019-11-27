[TOC]

## 1. 示例

```ruby
require 'fastlane'

Fastlane::FastFile.new.parse("lane :test do
  UI.message('hello')
end").runner.execute(:test)
```

```
 ~/Desktop  ruby main.rb
[11:15:16]: Driving the lane 'test' 🚀
[11:15:16]: hello
```



## 2. fastlane plugin spec

```ruby
require 'spec_helper'
require 'webmock/rspec'

describe Fastlane::Actions::CiBuildNumberAction do
  describe "xxx" do
    it "xxx" do
      ENV['TRAVIS'] = '1'
      ENV['TRAVIS_BUILD_NUMBER'] = '42'

      result = Fastlane::FastFile.new.parse("lane :test do
        ci_build_number
      end").runner.execute(:test)

      expect(result).to eq('42')

      ENV.delete('TRAVIS')
      ENV.delete('TRAVIS_BUILD_NUMBER')
    end
  end
end
```