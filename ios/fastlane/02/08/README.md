[TOC]



# 使用 ==Fastlane Helper== 封装可重用代码

## 1. WORKSPACE/toolbox/fastlane/helper/eat.rb

```ruby
require 'fastlane_core/ui/ui'

module Fastlane
  UI = FastlaneCore::UI unless Fastlane.const_defined?("UI")
  module Helper
    class EatHelper
      def self.show_message
        UI.message("[helper] [eat] .......")
      end
    end
  end
end
```

## 2. WORKSPACE/toolbox/fastlane/helper/happy.rb

```ruby
require 'fastlane_core/ui/ui'

module Fastlane
  UI = FastlaneCore::UI unless Fastlane.const_defined?("UI")
  module Helper
    class HappyHelper
      def self.show_message
        UI.message("[helper] [happy] .......")
      end
    end
  end
end
```

## 3. WORKSPACE/toolbox/fastlane/helper/cry.rb

```ruby
require 'fastlane_core/ui/ui'

module Fastlane
  UI = FastlaneCore::UI unless Fastlane.const_defined?("UI")
  module Helper
    class CryHelper
      def self.show_message
        UI.message("[helper] [cry] .......")
      end
    end
  end
end
```

## 4. WORKSPACE/toolbox/fastlane/Fastfile

```ruby
require_relative 'helper/eat.rb'
require_relative 'helper/happy.rb'
require_relative 'helper/cry.rb'

lane :test do
  Helper::EatHelper::show_message
  # Helper::WalkHelper::show_message
  Helper::HappyHelper::show_message
  Helper::CryHelper::show_message
end
```

## 5. bundle exec fastlane test

```ruby
 ~/WORKSPACE/toolbox   master ●  bundle exec fastlane test
[✔] 🚀

[00:13:45]: Driving the lane 'test' 🚀
[00:13:45]: [helper] [eat] .......
[00:13:45]: [helper] [happy] .......
[00:13:45]: [helper] [cry] .......
[00:13:45]: fastlane.tools finished successfully 🎉
```

## 6. 总结

都按照 **helper** 执行的.