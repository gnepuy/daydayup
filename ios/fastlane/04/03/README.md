[TOC]



## 1. Boolean

> 如下代码是重写 action 中的 self.available_options 方法, 如下例子都是

```ruby
FastlaneCore::ConfigItem.new(
  key: :commit,
  env_name: "MY_NEW_ACTION_COMMIT",
  description: "Commit the results if true",
  optional: true,
  default_value: false,
  is_string: false #=> specifying Boolean parameters, use is_string: false
)
```

执行 action 传递参数

```
MY_NEW_ACTION_COMMIT=true
MY_NEW_ACTION_COMMIT=false
MY_NEW_ACTION_COMMIT=yes
MY_NEW_ACTION_COMMIT=no
```



## 2. Array 

```ruby
FastlaneCore::ConfigItem.new(
  key: :files,
  env_name: "MY_NEW_ACTION_FILES",
  description: "One or more files to operate on",
  type: Array, #=> array
  optional: false
)
```

**Fastfile** 中调用 action 传递参数

```
my_new_action(files: "file1.txt,file2.txt") #=> ["file1.txt", "file2.txt"]
my_new_action(files: "file.txt") #=> ["file.txt"]
```

**命令行** run action 传递参数

```
export MY_NEW_ACTION_FILES=file1.txt,file2.txt
fastlane run <action>
```



## 3. Polymorphic (多态)

### 1. 我的 fastlane/actions/my_action.rb

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_CUSTOM_VALUE = :MY_ACTION_CUSTOM_VALUE
    end

    require 'pp'

    class MyActionAction < Action
      def self.run(params)
        UI.message("self.run: polymorphic_option:")
        pp params[:polymorphic_option]
      end

      ########################
      # @!group Documentation
      ########################

      def self.description
        "A short description with <= 80 characters of what this action does"
      end

      def self.details
        # Optional:
        # this is your chance to provide a more detailed description of this action
        "You can use this action to do cool things..."
      end

      def self.available_options
        [
          FastlaneCore::ConfigItem.new(
            key: :polymorphic_option,
            is_string: false,
            verify_block: ->(value) { verify_option(value) }
          )
        ]
      end

      def self.verify_option(value)
        UI.message("self.verify_option: value:")
        pp value.class
        pp value

        case value
        when String
          @polymorphic_option = value
        when Array
          @polymorphic_option = value.join(" ")
        when Hash
          @polymorphic_option = value.to_s
        else
          UI.user_error! "Invalid option: #{value.inspect}"
        end
      end

      def self.output
        [
          ['MY_ACTION_CUSTOM_VALUE', 'A description of what this value contains']
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

### 2. fastlane run action

测试发现这种方式参数，不能使用 **命令行** 执行 action.

### 3. Fastfile 中调用 action

```
lane :lan1 do
  my_action(polymorphic_option: "1,2,3,4,5,6")
  my_action(polymorphic_option: [1,2,3,4,5,6])
  my_action(polymorphic_option: {name: 'xiongzenghui', age: 99})
end
```

### 4. fastlane lan

```
 ~/collect_xxx/toolbox   master ●  bundle exec fastlane lan1
[✔] 🚀
[01:38:49]: Driving the lane 'lan1' 🚀
[01:38:49]: ----------------
[01:38:49]: --- Step: my ---
[01:38:49]: ----------------
[01:38:49]: self.verify_option: value:
String
"1,2,3,4,5,6"
[01:38:49]: self.run: polymorphic_option:
"1,2,3,4,5,6"
[01:38:49]: ----------------
[01:38:49]: --- Step: my ---
[01:38:49]: ----------------
[01:38:49]: self.verify_option: value:
Array
[1, 2, 3, 4, 5, 6]
[01:38:49]: self.run: polymorphic_option:
[1, 2, 3, 4, 5, 6]
[01:38:49]: ----------------
[01:38:49]: --- Step: my ---
[01:38:49]: ----------------
[01:38:49]: self.verify_option: value:
Hash
{:name=>"xiongzenghui", :age=>99}
[01:38:49]: self.run: polymorphic_option:
{:name=>"xiongzenghui", :age=>99}

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | my     | 0           |
| 2    | my     | 0           |
| 3    | my     | 0           |
+------+--------+-------------+

[01:38:49]: fastlane.tools finished successfully 🎉
```

ok.



## 4. Callback (回调) 

### 1. 我的 fastlane/actions/my_action.rb

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_CUSTOM_VALUE = :MY_ACTION_CUSTOM_VALUE
    end

    require 'pp'

    class MyActionAction < Action
      def self.run(params)
        params[:callback].call('value from MyActionAction') if params[:callback]
      end

      ########################
      # @!group Documentation
      ########################

      def self.description
        "A short description with <= 80 characters of what this action does"
      end

      def self.details
        # Optional:
        # this is your chance to provide a more detailed description of this action
        "You can use this action to do cool things..."
      end

      def self.available_options
        [
          FastlaneCore::ConfigItem.new(
            key: :callback,
            description: "Optional callback argument",
            optional: true,
            type: Proc
          )
        ]
      end

      def self.output
        [
          ['MY_ACTION_CUSTOM_VALUE', 'A description of what this value contains']
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

### 2. Fastfile 中调用 action

```ruby
lane :lan2 do
  callback = lambda do |result|
    UI.message("result = #{result}")
  end
  
  my_action(callback: callback)
end
```

### 3. fastlane lan

```
 ~/collect_xxx/toolbox   master ●  bundle exec fastlane lan2
[✔] 🚀
[01:44:07]: Driving the lane 'lan2' 🚀
[01:44:07]: ----------------
[01:44:07]: --- Step: my ---
[01:44:07]: ----------------
[01:44:07]: result = value from MyActionAction

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | my     | 0           |
+------+--------+-------------+

[01:44:07]: fastlane.tools finished successfully 🎉
```



## 5. action 参数 ==校验==

```ruby
verify_block = lambda do |value|
  # Has to be a String to get this far
  uri = URI(value)
  UI.error "Invalid scheme #{uri.scheme}" unless uri.scheme == "http" || uri.scheme == "https"
end

FastlaneCore::ConfigItem.new(
  key: :url,
  type: String,
  verify_block: verify_block
)
```



## 6. Conflicting (冲突)

### 1. 我的 fastlane/actions/my_action.rb

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_CUSTOM_VALUE = :MY_ACTION_CUSTOM_VALUE
    end

    require 'pp'

    class MyActionAction < Action
      def self.run(params)
      end

      ########################
      # @!group Documentation
      ########################

      def self.description
        "A short description with <= 80 characters of what this action does"
      end

      def self.details
        # Optional:
        # this is your chance to provide a more detailed description of this action
        "You can use this action to do cool things..."
      end

      def self.available_options
        [
          FastlaneCore::ConfigItem.new(
            key: :text,
            type: String,
            optional: true,
            conflicting_options: [:text_file]
          ),
          FastlaneCore::ConfigItem.new(
            key: :text_file,
            type: String,
            optional: true,
            conflicting_options: [:text]
          )
        ]
      end

      def self.output
        [
          ['MY_ACTION_CUSTOM_VALUE', 'A description of what this value contains']
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

### 2. Fastfile 中调用 action

```
lane :lan do
  my_action(text: '111111')
  my_action(text_file: '222222')
  my_action(text: '33333', text_file: '444444')
end
```

### 3. fastlane lane

```
 ~/collect_xxx/toolbox   master ●  bundle exec fastlane lan
[✔] 🚀
[01:49:20]: Driving the lane 'lan' 🚀
[01:49:20]: ----------------
[01:49:20]: --- Step: my ---
[01:49:20]: ----------------
[01:49:20]: ----------------
[01:49:20]: --- Step: my ---
[01:49:20]: ----------------
[01:49:20]: ----------------
[01:49:20]: --- Step: my ---
[01:49:20]: ----------------
[01:49:20]: You passed invalid parameters to 'my'.
[01:49:20]: Check out the error below and available options by running `fastlane action my`
+---------------+-----+
|    Lane Context     |
+---------------+-----+
| PLATFORM_NAME |     |
| LANE_NAME     | lan |
+---------------+-----+
[01:49:20]: Unresolved conflict between options: 'text' and 'text_file'

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | my     | 0           |
| 2    | my     | 0           |
| 💥   | my     | 0           |
+------+--------+-------------+

[01:49:20]: fastlane finished with errors

[!] Unresolved conflict between options: 'text' and 'text_file'
```

- 前两次调用 action 是 ok 的
- 最后一次 **同时传递** test 和 text_file 参数时产生 **冲突**



## 7. Optional (可选) & Default (默认值)

```ruby
FastlaneCore::ConfigItem.new(
  key: :build_configuration,
  description: "Which build configuration to use",
  type: String,
  optional: true, #=> 可选
  default_value: "Release" #=> 默认值
)
```
