[TOC]



## 1. Action 直接 return value

### 1. my_action

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_KEY_1 = :MY_ACTION_KEY_1
      MY_ACTION_KEY_2 = :MY_ACTION_KEY_2
      MY_ACTION_KEY_3 = :MY_ACTION_KEY_3
    end

    require 'pp'

    class MyActionAction < Action
      def self.run(params)
        # 直接返回需要的【返回值】即可
        return "wo cao ni ma ~"
      end

      ####################################
      # @!group Documentation
      ####################################

      def self.description
        "A short description with <= 80 characters of what this action does"
      end

      def self.details
        "You can use this action to do cool things..."
      end
      
      def self.return_type
        :stirng
      end

      def self.return_value
        'return a name for you ~'
      end
    end
  end
end
```

### 2. Fastfile 调用 my_action 并直接获取 返回值

```ruby
lane :build do
  ret = my_action
  UI.message("ret = #{ret}")
end
```

### 3. fastlane exec

```
# bundle exec fastlane build
[✔] 🚀
[13:19:00]: Driving the lane 'build' 🚀
[13:19:00]: ----------------
[13:19:00]: --- Step: my ---
[13:19:00]: ----------------
[13:19:00]: ret = wo cao ni ma ~

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | my     | 0           |
+------+--------+-------------+

[13:19:00]: fastlane.tools finished successfully
```



## 2. module SharedValues

### 1. 参考 git_branch

https://github.com/fastlane/fastlane/blob/master/fastlane/lib/fastlane/actions/git_branch.rb

### 2. my_action

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_KEY_1 = :MY_ACTION_KEY_1
      MY_ACTION_KEY_2 = :MY_ACTION_KEY_2
      MY_ACTION_KEY_3 = :MY_ACTION_KEY_3
    end

    require 'pp'

    class MyActionAction < Action
      def self.run(params)
        self.lane_context[SharedValues::MY_ACTION_KEY_1] = 'MY_ACTION_KEY_1_VALUE'
        self.lane_context[SharedValues::MY_ACTION_KEY_2] = 'MY_ACTION_KEY_2_VALUE'
        self.lane_context[SharedValues::MY_ACTION_KEY_3] = 'MY_ACTION_KEY_3_VALUE'
      end

      ####################################
      # @!group Documentation
      ####################################

      def self.description
        "A short description with <= 80 characters of what this action does"
      end

      def self.details
        "You can use this action to do cool things..."
      end

      def self.available_options
        nil
      end

      def self.output
        [
          ['MY_ACTION_KEY_1', 'A description of MY_ACTION_KEY_1'],
          ['MY_ACTION_KEY_2', 'A description of MY_ACTION_KEY_2'],
          ['MY_ACTION_KEY_3', 'A description of MY_ACTION_KEY_3']
        ]
      end

      def self.return_value
        # If your method provides a return value, you can describe here what it does
      end

      def self.authors
        ["Your GitHub/Twitter Name"]
      end

      def self.is_supported?(platform)
        platform == :ios
      end
    end
  end
end
```

### 3. Fastfile 调用 my_action 并从 shared values 中获取 返回值

```ruby
lane :build do
  # 1.
  my_action

  # 2. 获取 action 存储到 SharedValues 中的返回值
  UI.message(self.lane_context[SharedValues::MY_ACTION_KEY_1])
  UI.message(self.lane_context[SharedValues::MY_ACTION_KEY_2])
  UI.message(self.lane_context[SharedValues::MY_ACTION_KEY_3])
end
```

### 4. fastlane exec

```
 ~/collect_xxx/toolbox   master ●  bundle exec fastlane build
[✔] 🚀
[13:09:24]: Driving the lane 'build' 🚀
[13:09:24]: ----------------
[13:09:24]: --- Step: my ---
[13:09:24]: ----------------
[13:09:24]: MY_ACTION_KEY_1_VALUE
[13:09:24]: MY_ACTION_KEY_2_VALUE
[13:09:24]: MY_ACTION_KEY_3_VALUE

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | my     | 0           |
+------+--------+-------------+

[13:09:24]: fastlane.tools finished successfully
```
