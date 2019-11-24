[TOC]



## 1. 在 fastlane 项目根目录下, 创建一个 action

```
->  fastlane new_action
[14:02:49]: Get started using a Gemfile for fastlane https://docs.fastlane.tools/getting-started/ios/setup/#use-a-gemfile
Must be lower case, and use a '_' between words. Do not use '.'
examples: 'testflight', 'upload_to_s3'
[14:02:52]: Name of your action:
```

提示输入 action 的名字.



## 2. 输入要添加的 action 名字: git_remove_tag

```
[14:02:52]: Name of your action: git_remove_tag
[14:03:41]: Created new action file './fastlane/actions/git_remove_tag.rb'. Edit it to implement your custom action.
->
```



## 3. 实现 git_remove_tag.rb 源文件

按照需求重写如上几个函数完成 **git remove tag** 功能的自定义action

```ruby
module Fastlane
  module Actions
    module SharedValues
      GIT_REMOVE_TAG_CUSTOM_VALUE = :GIT_REMOVE_TAG_CUSTOM_VALUE
    end

    class GitRemoveTagAction < Action
      def self.run(params)
          # 1、获取所有输入的参数值
          tageName = params[:tag] # 参数1、tag
          isRemoveLocationTag = params[:isRL] # 参数2、是否需要删除本地标签
          isRemoveRemoteTag = params[:isRR] # 参数3、是否需要删除远程标签

          # 2、定义一个数组，拼接所有要执行的shell命令
          cmds = [
            ("git tag -d #{tageName}" if isRemoveLocationTag), # 删除【本地】git库的标签 
            ("git push origin :#{tageName}" if isRemoveRemoteTag), # 删除【远程】git库的标签 
          ].compact() # compact() 过滤掉数组中的nil空值

          # 3、执行数组里面的所有的命令
          result = Actions.sh(cmds.join('&'))
          UI.message("执行完毕 remove_tag的操作 🚀")

          # 4. 返回结果值
          return result
      end

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
          # 默认值函数定义
          # def func(age:0, name:"none", addr:"none")
          #   print("age = ", age, "\n")
          #   print("name = ", name, "\n")
          #   print("addr = ", addr, "\n")
          # end
          #
          # 调用函数时，指定形参的值，可以打乱形参的顺序
          # func age:19, name:"xiong", addr:"hunan"
          #
          # 如下创建action所需形参的函数原型
          # def FastlaneCore::ConfigItem.new(key,description,optional,is_string,default_value)
          #  ...
          # end

          # 参数1. tag值的参数描述，不可以忽略<必须输入>，字符串类型，没有默认值
          FastlaneCore::ConfigItem.new(
            key: :tag, #形参key的参数值为 :tag（symbol对象）
            description: "tag 号是多少",
            optional:false,# 是不是可以省略
            is_string: true # true: 是不是字符串
          ),

          # 参数2. 是否删除本地标签
          FastlaneCore::ConfigItem.new(
            key: :isRL,
            description: "是否删除本地标签",
            optional:true,# 是不是可以省略
            is_string: false, # true: 是不是字符串
            default_value: true
          ), 

          # 参数3. 是否删除远程标签
          FastlaneCore::ConfigItem.new(
            key: :isRR,
            description: "是否删除远程标签",
            optional:true,# 是不是可以省略
            is_string: false, # true: 是不是字符串
            default_value: true
          ) 
        ]
      end

      def self.output
        [
          ['GIT_REMOVE_TAG_CUSTOM_VALUE', 'A description of what this value contains']
        ]
      end

      def self.return_value
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



## 4. 验证 action 的实现 ruby 文件 是否合法

```
->  fastlane action git_remove_tag
[14:41:25]: Get started using a Gemfile for fastlane https://docs.fastlane.tools/getting-started/ios/setup/#use-a-gemfile
Loading documentation for git_remove_tag:

+--------------------------------------------------------------------+
|                           git_remove_tag                           |
+--------------------------------------------------------------------+
| A short description with <= 80 characters of what this action does |
|                                                                    |
| You can use this action to do cool things...                       |
|                                                                    |
| Created by Your GitHub/Twitter Name                                |
+--------------------------------------------------------------------+

+------+------------------+---------+---------+
|           git_remove_tag Options            |
+------+------------------+---------+---------+
| Key  | Description      | Env Var | Default |
+------+------------------+---------+---------+
| tag  | tag 号是多少     |         |         |
| isRL | 是否删除本地标签 |         | true    |
| isRR | 是否删除远程标签 |         | true    |
+------+------------------+---------+---------+

+-----------------------------+-------------------------------------------+
|                     git_remove_tag Output Variables                     |
+-----------------------------+-------------------------------------------+
| Key                         | Description                               |
+-----------------------------+-------------------------------------------+
| GIT_REMOVE_TAG_CUSTOM_VALUE | A description of what this value contains |
+-----------------------------+-------------------------------------------+
Access the output values using `lane_context[SharedValues::VARIABLE_NAME]`

More information can be found on https://docs.fastlane.tools/actions

->
```

输出如上表格信息表示 ok。



## 5. 使用 action

### 1. `Fastfile#lane` 调用 action

```ruby
lane xxxxx: do |options|
  tagName = options[:tag]

  # 如果存在该tag则删除
  if git_tag_exists(tag: tagName)
    UI.message("发现 tag:#{tagName} 存在，即将执行删除动作 🚀") # log输出
    git_remove_tag(tag:tagName, isRL:true, isRR:true) # 不使用缺省的默认值参数
    # git_remove_tag(tag:tagName) # 使用缺省的默认值参数
  end
end
```

### 2. fastlane 直接执行 action

```
fastlane XZHNetwork tag:0.1.8 specName:DownLoader repo:PrivateSpecsGather
```

### 3. rspec 中运行 action

```ruby
describe Fastlane::Actions::MakeAction do
  describe '#run' do
    it 'prints a message' do
      expect(Fastlane::UI).to receive(:message).with("The make plugin is working!")

      Fastlane::Actions::MakeAction.run(nil)
    end
  end
end
```



## 6. fastlane run action 传入参数

### 1. fastalne 执行 action

```
fastlane run <action名字> <key1:value1> <key2:value2> ... <keyN:valueN>
```

### 2. Fastfile lane 调用 action

```ruby
git_commit(
  git: 'xxxx',
  branch: 'xxxxx',
  tag: '0.0.1'
)
```

### 3. env

```ruby
module Fastlane
  module Actions
    module SharedValues
      MY_ACTION_CUSTOM_VALUE = :MY_ACTION_CUSTOM_VALUE
    end

    class MyActionAction < Action

      ................


      def self.available_options
        [
          FastlaneCore::ConfigItem.new(key: :api_token,
                                       env_name: "FL_MY_ACTION_API_TOKEN", # The name of the environment variable
                                       description: "API Token for MyActionAction", # a short description of this parameter
                                       verify_block: proc do |value|
                                          UI.user_error!("No API token for MyActionAction given, pass using `api_token: 'token'`") unless (value and not value.empty?)
                                          # UI.user_error!("Couldn't find file at path '#{value}'") unless File.exist?(value)
                                       end),
          FastlaneCore::ConfigItem.new(key: :development,
                                       env_name: "FL_MY_ACTION_DEVELOPMENT",
                                       description: "Create a development certificate instead of a distribution one",
                                       is_string: false, # true: verifies the input is a string, false: every kind of value
                                       default_value: false) # the default value if the user didn't provide one
        ]
      end

      ................
    end
  end
end
```

通过 env 传递参数

```ruby
ENV['FL_MY_ACTION_API_TOKEN'] = ....
ENV['FL_MY_ACTION_DEVELOPMENT'] = ....
```


