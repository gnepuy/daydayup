[TOC]



## 1. ==开发== 一个 fastlane plugin

### 1. 在 fastlane 项目 (toolbox) 根目录下, 创建一个 plugin

```
fastlane new_plugin [plugin_name]
```

比如

```
fastlane new_plugin haha
```

输出如下

```
...............................................
...............................................
...............................................

Your plugin was successfully generated at fastlane-plugin-haha/ 🚀

To get started with using this plugin, run

    fastlane add_plugin haha

from a fastlane-enabled app project directory and provide the following as the path:

    /Users/xiongzenghui/collect_xxx/toolbox/fastlane-plugin-haha
```

提示 plugin 的开发目录是 `toolbox/fastlane-plugin-haha/` 这个目录.

### 2. toolbox 目录

```
 ~/collect_xxx/toolbox   master ●  tree
.
├── Gemfile
├── Gemfile.lock
├── README.md
├── fastlane
│   ├── Appfile
│   ├── Fastfile
│   ├── README.md
│   ├── actions
│   │   ├── git_archive_install.rb
│   │   ├── jenkins_mr_build.rb
│   │   └── my_action.rb
│   ├── dayly_job
│   │   └── dayly_Fastfile.rb
│   ├── module_job
│   │   └── module_Fastfile.rb
│   ├── mr_job
│   │   └── mr_Fastfile.rb
│   ├── report.xml
│   └── tools
├── fastlane-plugin-haha
│   ├── Gemfile
│   ├── LICENSE
│   ├── README.md
│   ├── Rakefile
│   ├── fastlane
│   │   ├── Fastfile
│   │   └── Pluginfile
│   ├── fastlane-plugin-haha.gemspec
│   ├── lib
│   │   └── fastlane
│   │       └── plugin
│   │           ├── haha
│   │           │   ├── actions
│   │           │   │   └── haha_action.rb
│   │           │   ├── helper
│   │           │   │   └── haha_helper.rb
│   │           │   └── version.rb
│   │           └── haha.rb
│   └── spec
│       ├── haha_action_spec.rb
│       └── spec_helper.rb
└── jenkins_mr.rb
```

- 可以看到会多出一个 **fastlane-plugin-haha** 的目录
- **fastlane-plugin-haha** 这个目录就是后续用来开发 plugin 的源码目录
- 开发过程就是开发一个 **gem** 

### 3. toolbox/fastlane-plugin-haha/ ==plugin== 目录

```
 ~/collect_xxx/toolbox/fastlane-plugin-haha   master ●  tree
.
├── Gemfile
├── LICENSE
├── README.md
├── Rakefile
├── fastlane
│   ├── Fastfile
│   └── Pluginfile
├── fastlane-plugin-haha.gemspec
├── lib
│   └── fastlane
│       └── plugin
│           ├── haha
│           │   ├── actions
│           │   │   └── haha_action.rb
│           │   ├── helper
│           │   │   └── haha_helper.rb
│           │   └── version.rb
│           └── haha.rb
└── spec
    ├── haha_action_spec.rb
    └── spec_helper.rb
```

可以看到这个 **plugin** 的目录结构, 其实就是一个 **ruby gem** 的目录结构.

### 4. toolbox/fastlane-plugin-haha/lib/fastlane/plugin/haha/actions/haha_action.rb

```ruby
require 'fastlane/action'
require_relative '../helper/haha_helper'

module Fastlane
  module Actions
    class HahaAction < Action
      def self.run(params)
        UI.message("The haha plugin is working!")
      end

      def self.description
        "this is a demo plugin"
      end

      def self.authors
        ["xiongzenghui"]
      end

      def self.return_value
        # If your method provides a return value, you can describe here what it does
      end

      def self.details
        # Optional:
        "this is a demo plugin this is a demo plugin this is a demo plugin this is a demo plugin"
      end

      def self.available_options
        [
          # FastlaneCore::ConfigItem.new(key: :your_option,
          #                         env_name: "HAHA_YOUR_OPTION",
          #                      description: "A description of your option",
          #                         optional: false,
          #                             type: String)
        ]
      end

      def self.is_supported?(platform)
        # Adjust this if your plugin only works for a particular platform (iOS vs. Android, for example)
        # See: https://docs.fastlane.tools/advanced/#control-configuration-by-lane-and-by-platform
        #
        # [:ios, :mac, :android].include?(platform)
        true
      end
    end
  end
end
```

- 这个 **plugin/haha/actions/haha_action.rb** 其实就跟之前 **实现 action** 是一样的写法.
- 所以如果是需要把之前的 **action** 转换成 **plugin** 的话，直接把代码 **拷贝** 过来就行



## 2. toolbox/fastlane-plugin-haha/ 目录下, fastlane test ==测试== plugin

在 **toolbox/fastlane-plugin-haha/ 目录下** 执行 **bundle exec fastlane test**

```
 ~/collect_xxx/toolbox/fastlane-plugin-haha   master ●  bundle exec fastlane test
[✔] 🚀
+----------------------+---------+--------+
|              Used plugins               |
+----------------------+---------+--------+
| Plugin               | Version | Action |
+----------------------+---------+--------+
| fastlane-plugin-haha | 0.1.0   | haha   |
+----------------------+---------+--------+

[02:43:02]: Driving the lane 'test' 🚀
[02:43:02]: ------------------
[02:43:02]: --- Step: haha ---
[02:43:02]: ------------------
[02:43:02]: The haha plugin is working!

+------+--------+-------------+
|      fastlane summary       |
+------+--------+-------------+
| Step | Action | Time (in s) |
+------+--------+-------------+
| 1    | haha   | 0           |
+------+--------+-------------+

[02:43:02]: fastlane.tools finished successfully 🎉
```

plugin 正常运行.



## 3. fastlane 项目中 ==注册== 要使用的 plugin

### 1. toolbox/Gemfile : 指定 Pluginfile 文件的位置

```ruby
source "https://rubygems.org"

# gem "activesupport", '2.16.3'
gem "cocoapods", '1.6.0'
gem "fastlane"
gem "dotenv"
gem "rake"
gem "rspec"

############################## 加载 plugin ##############################
plugins_path = File.join(File.dirname(__FILE__), 'fastlane', 'Pluginfile')
eval_gemfile(plugins_path) if File.exist?(plugins_path)
```

### 2. toolbox/fastlane/Pluginfile : 注册要使用的 plugin

- 1) 已经发布到 **rubygems.org** 主仓库服务器中的 plugin
- 2) 指定某个 **remote git repo** 中的 plugin
- 3) 指定某个 **local path** 中的 plugin

```ruby
# Autogenerated by fastlane
#
# Ensure this file is checked in to source control!

######################################################
## 1、引用【发布到 fastlane plugin repo】中的 plugin
gem 'fastlane-plugin-pgyer'

######################################################
## 2、引用【本地】中的 plugin
gem "fastlane-plugin-haha", path: "../fastlane-plugin-haha"
# gem 'fastlane-plugin-local-插件1', path: '"../fastlane-plugins-local1'
# gem 'fastlane-plugin-local-插件2', path: '"../fastlane-plugins-local2'
# gem 'fastlane-plugin-local-插件3', path: '"../fastlane-plugins-local3'
# gem 'fastlane-plugin-local-插件4', path: '"../fastlane-plugins-local4'

######################################################
## 3、引用【remote git repo】中的 plugin
# gem 'fastlane-plugin-remote-插件1', git: 'https://github.com/xiongzenghui/fastlane-plugin-remote1.git'
# gem 'fastlane-plugin-remote-插件2', git: 'https://github.com/xiongzenghui/fastlane-plugin-remote2.git'
# gem 'fastlane-plugin-remote-插件3', git: 'https://github.com/xiongzenghui/fastlane-plugin-remote3.git'
# gem 'fastlane-plugin-remote-插件4', git: 'https://github.com/xiongzenghui/fastlane-plugin-remote4.git'
```

### 3. 在 fastlane 项目根目录下 bundle install 

之后就可以愉快的使用注册过的 plugin 了..




## 4. ==发布== plugin 到 ==RubyGems 服务器==

> 其实就是将 plugin 这个 ruby gem 项目，发布到 **https://rubygems.org** 主仓库中.

### 1. 在 rubygems.org 官网注册账号

![](01.png)

会得到账号和密码.

### 2. 把 plugin/ 推送到 remote repo

> 注意: 修改 fastlane-plugin-[plugin_name].gemspec 文件 的 spec.homepage 指向远程仓库的 repo

我选的是 github

### 3. plugin/ 执行如下三步

```
bundle install
rake install
rake release
```

```
#fastlane-plugins/fastlane-plugin-git_clone   master  rake release
fastlane-plugin-git_clone 0.1.0 built to pkg/fastlane-plugin-git_clone-0.1.0.gem.
Tagged v0.1.0.
Pushed git commits and tags.
rake aborted!
Your rubygems.org credentials aren't set. Run `gem push` to set them.
/Users/xiongzenghui/.rvm/gems/ruby-2.4.1/gems/rake-12.3.2/exe/rake:27:in `<top (required)>'
Tasks: TOP => release => release:rubygem_push
(See full trace by running task with --trace)
```

- 会构建生成 **gem 包** : `pkg/fastlane-plugin-git_clone-0.1.0.gem`
- 第一次提交会错误提示: Your rubygems.org credentials aren't set. Run `gem push` to set them.

### 4. gem push pkg/fastlane-plugin-git_clone-0.1.0.gem

#### 1. 要求输入 rubygems.org 注册的 email 和 password

![](02.png)

#### 2. 成功提交 gem

```
Enter your RubyGems.org credentials.
Don't have an account yet? Create one at https://rubygems.org/sign_up
   Email:   zxcvb1234001@163.com
Password:

Signed in.
Pushing gem to https://rubygems.org...
Successfully registered gem: fastlane-plugin-git_clone (0.1.0)
```

### 5. rubygems.org 查询提交的 gem

> https://rubygems.org/gems/fastlane-plugin-git_clone

![](03.png)

### 6. 如果需要删除 rubygems.org 已经存在的 version

```
gem yank fastlane-plugin-git_clone -v 0.1.0
```

### 7. 后续更新新的 version gem 直接三步完成

```
bundle install
rake build
rake release
```

比如:

```
............
 ~/collect_rubygems/fastlane-plugins/fastlane-plugin-git_clone   master  rake release
fastlane-plugin-git_clone 0.1.1 built to pkg/fastlane-plugin-git_clone-0.1.1.gem.
Tagged v0.1.1.
Pushed git commits and tags.
Pushed fastlane-plugin-git_clone 0.1.1 to rubygems.org
```



## 5. 在其他的 fastlane app 项目中, 使用已经发布到 rubygems.org 中的 plugin

> 添加插件的【引用】: fastlane add_plugin git_clone

```
 ~/collect_rubygems/fastlane-plugins/fastlane-plugin-git_clone   master  fastlane add_plugin git_clone
[✔] 🚀
[11:53:15]: fastlane detected a Gemfile in the current directory
[11:53:15]: however it seems like you don't use `bundle exec`
[11:53:15]: to launch fastlane faster, please use
[11:53:15]:
[11:53:15]: $ bundle exec fastlane add_plugin git_clone
[11:53:15]:
[11:53:15]: Get started using a Gemfile for fastlane https://docs.fastlane.tools/getting-started/ios/setup/#use-a-gemfile
+---------------------------+---------+-----------+
|                  Used plugins                   |
+---------------------------+---------+-----------+
| Plugin                    | Version | Action    |
+---------------------------+---------+-----------+
| fastlane-plugin-git_clone | 0.1.1   | git_clone |
+---------------------------+---------+-----------+

[11:53:16]: Make sure to commit your Gemfile, Gemfile.lock and Pluginfile to version control
Installing plugin dependencies...
Successfully installed plugins
```



## 6. 查看 plugin 参数的详细信息 (其实查看 action)

- 1) 如果查看 **fastlane-plugin-xx_yy** 的使用
- 2) 其实查看 plugin 对应的 **action** 的使用: **fastlane action xx_yy**

```
# fastlane action git_clone
[✔] 🚀
[14:14:36]: fastlane detected a Gemfile in the current directory
[14:14:36]: however it seems like you don't use `bundle exec`
[14:14:36]: to launch fastlane faster, please use
[14:14:36]:
[14:14:36]: $ bundle exec fastlane action git_clone
[14:14:36]:
[14:14:36]: Get started using a Gemfile for fastlane https://docs.fastlane.tools/getting-started/ios/setup/#use-a-gemfile
+---------------------------+---------+-----------+
|                  Used plugins                   |
+---------------------------+---------+-----------+
| Plugin                    | Version | Action    |
+---------------------------+---------+-----------+
| fastlane-plugin-git_clone | 0.1.1   | git_clone |
+---------------------------+---------+-----------+

Loading documentation for git_clone:

+------------------------------------------------------------------------------------+
|                                     git_clone                                      |
+------------------------------------------------------------------------------------+
| this is a wrapper for git clone command                                            |
|                                                                                    |
| exec like this: git clone xxx.git -b master /path/to/xxx --single-branch --depth=1 |
|                                                                                    |
| Created by xiongzenghui                                                            |
+------------------------------------------------------------------------------------+

+---------------+--------------------+----------------------------+---------+
|                             git_clone Options                             |
+---------------+--------------------+----------------------------+---------+
| Key           | Description        | Env Var                    | Default |
+---------------+--------------------+----------------------------+---------+
| git           | where from clone   | FL_GIT_CLONE_GIT           |         |
| path          | where clone to dir | FL_GIT_CLONE_PATH          |         |
| depth         | --depth=1          | FL_GIT_CLONE_DEPTH         |         |
| branch        | -b master          | FL_GIT_CLONE_BRANCH        |         |
| single_branch | --single-branch    | FL_GIT_CLONE_SINGLE_BRANCH | false   |
+---------------+--------------------+----------------------------+---------+
* = default value is dependent on the user's system

+-------------------------------------------------------+
|                git_clone Return Value                 |
+-------------------------------------------------------+
| return a status (boolean) of git clone sh exec result |
+-------------------------------------------------------+

More information can be found on https://docs.fastlane.tools/actions/git_clone
```



## 7. 总结 action 对比 plugin

- 1) 如果 **单个项目** 简单使用 **action** 即可起到 **代码封装重用**

- 2) 果是 **多个项目** 之间需要 **共享 action** , 就需要使用 **plugins**

