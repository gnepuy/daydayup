[TOC]




## 1. fastlane action 是什么？干什么？

- 之前我们一直是通过编写 **Fastfile** 实现一个功能的自动化

- 但是如果所有的代码都 **堆积在一个 Fastfile** 文件中，会造成代码臃肿难以维护

- 所以 fastlane 提供 action 这样的东西，来让你把一些可重用的代码隔离封装起来



## 2. 查看某个 action 的用法

比如查看 **cocoapods** 这个 action 怎么用:

```
fastlane action cocoapods
```



## 3. `Fastfile#lane` 与 action 工作目录 ==不一样==

### 1. Fastfile#lane 工作目录: `$WORKSPACE/fastlane/`

```ruby
sh "pwd" # => "$WORKSPACE/fastlane"
puts Dir.pwd # => "$WORKSPACE/fastlane"

lane :something do
  sh "pwd" # => "$WORKSPACE/fastlane"
  puts Dir.pwd # => "$WORKSPACE/fastlane"

  my_action
end
```

注意: while all user code **from the Fastfile** runs **inside** the  `./fastlane directory`.

### 2. action#run 工作目录: `$WORKSPACE/`

```ruby
module Fastlane
  module Actions
    class MyAction < Action
      def run(params)
        puts Dir.pwd # => "$WORKSPACE"
      end
    end
  end
end
```

注意: Notice how every **action** and every **plugin**'s code runs **in the root of the project**, 



## 4. 如果需要在 `$WORKSPACE/` 下执行的代码, 最好使用 ==action== 封装

比如 nicolas buil 代码，使用一个 action 封装起来 run 方法中执行

```ruby
module Fastlane
  module Actions
    class JenkinsMrBuildAction < Action
      def self.run(params)
        puts "[JenkinsMrBuildAction] pwd = #{Dir.pwd}"

        mrtitle = ENV['gitlabMergeRequestTitle'].to_s
        mrdesc = ENV['gitlabMergeRequestDescription'].to_s

        system("工程构建代码")
      end

      ...................
    end
  end
end
```

- 那么当这个 action 执行的时候，自动就会在 `$WORKSPACE/` 根目录下执行了
- 就不用在 Fastfile#lane 中各种 **Dir.chdir(xxx)** 操作



## 5. lane 与 action  ==工作目录== 示例

### 1. Fastfile

```ruby
lane :test do
  # 1.
  UI.message("lane :test 1: #{Dir.pwd}")

  # 2.
  my_action

  # 3. 会改变 工作目录
  Dir.chdir('../')

  # 4.
  UI.message("lane :test 4: #{Dir.pwd}")

  # 5.
  my_action

  # 6.
  UI.message("lane :test 6: #{Dir.pwd}")
end
```

### 2. action

```ruby
module Fastlane
  module Actions
    class MyActionAction < Action
      def self.run(params)
        UI.message("my_action: #{Dir.pwd}")
      end

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

### 3. WORKSPACE 根目录

目录内都是一个个的 **目录**

```
 ~/collect_workspace  ll
total 0
drwxr-xr-x   5 xiongzenghui  staff   160B  7  9 22:16 FAQ
drwxr-xr-x  26 xiongzenghui  staff   832B  6 11 16:47 arsenal
drwxr-xr-x  14 xiongzenghui  staff   448B  3 15 13:39 ios-ci
drwxr-xr-x  15 xiongzenghui  staff   480B  6 11 16:27 ios_build_env_setup
drwxr-xr-x  31 xiongzenghui  staff   992B  6 12 01:09 ios_projects
drwxr-xr-x  23 xiongzenghui  staff   736B  6 20 20:22 kanshan
drwxr-xr-x  14 xiongzenghui  staff   448B  6 27 15:55 metis
drwxr-xr-x  21 xiongzenghui  staff   672B  6 27 13:17 nicolas
drwxr-xr-x  27 xiongzenghui  staff   864B  6 25 10:00 nicolas-server
drwxr-xr-x  21 xiongzenghui  staff   672B  7  9 11:15 nicolas_self
drwxr-xr-x  14 xiongzenghui  staff   448B  7 10 13:47 toolbox
drwxr-xr-x  18 xiongzenghui  staff   576B  7  3 14:50 venom
```

定义环境变量表示 **WORKSPACE** 假设是我们的 **工作目录** 最顶层 目录

```
export WORKSPACE=/Users/xiongzenghui/collect_workspace
```
### 4. 进入 WORKSPACE 下的 **fastlane app 项目** 的根目录

```
# cd $WORKSPACE/toolbox
# ll
total 56
-rw-r--r--   1 xiongzenghui  staff   1.2K  7 10 12:48 Gemfile
-rw-r--r--   1 xiongzenghui  staff   7.5K  7 10 12:14 Gemfile.lock
-rw-r--r--   1 xiongzenghui  staff   328B  7 10 13:47 Makefile
-rw-r--r--@  1 xiongzenghui  staff   1.0K  7 10 19:33 README.md
drwxr-xr-x  12 xiongzenghui  staff   384B  7 10 16:31 fastlane
-rw-r--r--   1 xiongzenghui  staff   195B  7 10 13:23 jenkins_module.rb
-rw-r--r--   1 xiongzenghui  staff   191B  7 10 13:02 jenkins_mr.rb
```

### 5. 执行 Fastfile 中的 test lane

```
# bundle exec fastlane test
[✔] 🚀
+---------------------------+---------+-----------+
|                  Used plugins                   |
+---------------------------+---------+-----------+
| Plugin                    | Version | Action    |
+---------------------------+---------+-----------+
| fastlane-plugin-git_clone | 0.1.1   | git_clone |
+---------------------------+---------+-----------+

[23:48:42]: Driving the lane 'test' 🚀
[23:48:42]: lane :test 1: /Users/xiongzenghui/collect_workspace/toolbox/fastlane
[23:48:42]: ----------------
[23:48:42]: --- Step: my ---
[23:48:42]: ----------------
[23:48:42]: my_action: /Users/xiongzenghui/collect_workspace/toolbox
Fastfile:14: warning: conflicting chdir during another chdir block
[23:48:42]: lane :test 4: /Users/xiongzenghui/collect_workspace/toolbox
[23:48:42]: ----------------
[23:48:42]: --- Step: my ---
[23:48:42]: ----------------
[23:48:42]: my_action: /Users/xiongzenghui/collect_workspace
[23:48:42]: lane :test 6: /Users/xiongzenghui/collect_workspace/toolbox

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | my     | 0           |
| 2    | my     | 0           |
+------+--------+-------------+

[23:48:42]: fastlane.tools finished successfully 🎉
```

输出的 dir 目录依次:

- 1.1) lane :test 1: /Users/xiongzenghui/collect_workspace/toolbox/fastlane
- 1.2) my_action: /Users/xiongzenghui/collect_workspace/toolbox
- 2.1) lane :test 4: /Users/xiongzenghui/collect_workspace/toolbox
- 2.2) my_action: /Users/xiongzenghui/collect_workspace
- 2.3) lane :test 6: /Users/xiongzenghui/collect_workspace/toolbox

### 6. 结论

- 1) **lane** 默认执行目录是 `WORKSPACE/fastlaneApp项目/fastlane/`

- 2) **action** 默认执行目录是 `WORKSPACE/fastlaneApp项目/`

- 3) 而不管 **lane** 如何 **改变目录** , **action** 总是比 **lane** 要 **少一层** 目录路径

### 7. ~~==改变== 工作目录~~ (fastlane 不建议这么做)

- 1) This is important to consider when migrating **existing** code from **your Fastfile** into your own action or plugin

- 2) To **change the directory** manually you can use standard **Ruby blocks**:

```ruby
Dir.chdir("..") do
  # code here runs in the parent directory
end
```

- 1) This behavior **isn't great**
  - 这种方式不是特别好

- 2) and has been like this since the very early days of fastlane
  - FastLane的早期就一直如此

- 3) As much as we'd like to change it, there is no good way to do so,
  - 尽管我们很想改变它，但没有好的方法可以做到这一点

- 4) without breaking thousands of production setups, so we decided to keep it as is for now.
  - 如果不打破成千上万的生产设置，所以我们决定保持现状。

### 8. action 内部 ==路径== 管理总结  

- 1) 在 **action 内部** 不要随便 **改变** 工作目录

- 2) 如果一定要更改，则一定要 **先备份路径** ，在完成时再 **恢复路径**

- 3) 推荐 **Fastfile#lane** 调用 **action** 时候，传入的就是 **最终操作文件** 的 **绝对路径**



