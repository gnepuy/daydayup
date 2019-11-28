[TOC]



## 1. ==Fastfile== 不能使用 ==require_relative== 相对引用其他 ==本地 rb 文件==

### 1. ==remote== fastlane 项目

#### 1. 目录结构

```
 ~/Desktop/MyFastfiles   master  tree
.
├── Gemfile
├── Gemfile.lock
├── README.en.md
├── README.md
└── fastlane
    ├── Appfile
    ├── Fastfile
    ├── README.md
    ├── report.xml
    └── tool.rb

1 directory, 9 files
```

#### 2. fastlane/Fastfile

```ruby
require_relative 'tool'

lane :haha do
  log
end
```

#### 3. fastlane/tool.rb

```ruby
def log
  puts '🎉' * 50
end
```

### 2. ==local== fastlane 项目

#### 1. 目录结构

```
 ~/Desktop/FastlaneDemo   master ●  tree
.
├── Gemfile
├── Gemfile.lock
├── README.en.md
├── README.md
└── fastlane
    ├── Appfile
    ├── Fastfile
    ├── README.md
    └── report.xml

1 directory, 8 files
```

#### 2. fastlane/tool.rb

```ruby
import_from_git(
  url: 'git@gitee.com:garywade/MyFastfiles.git',
  path: 'fastlane/Fastfile'
)

lane :test do
end
```

#### 3. exec local Fastfile

```
 ~/Desktop/FastlaneDemo   master ●  bundle exec fastlane test
[✔] 🚀
[01:20:15]: -----------------------------
[01:20:15]: --- Step: import_from_git ---
[01:20:15]: -----------------------------
[01:20:15]: Cloning remote git repo...
[01:20:15]: $ git clone git@gitee.com:garywade/MyFastfiles.git /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git --depth 1 -n
[01:20:15]: ▸ Cloning into '/var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git'...
[01:20:16]: $ cd /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git && git checkout HEAD fastlane/Fastfile
[01:20:16]: $ cd /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git && git checkout HEAD fastlane/actions
[01:20:16]: ▸ error: pathspec 'fastlane/actions' did not match any file(s) known to git
bundler: failed to load command: fastlane (/Users/xiongzenghui/.rvm/gems/ruby-2.4.1/bin/fastlane)
LoadError: cannot load such file -- /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git/fastlane/tool
  ../../../../../var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git/fastlane/Fastfile:1:in `require_relative'
  ../../../../../var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git/fastlane/Fastfile:1:in `parsing_binding'
..........................................................................................
..........................................................................................
..........................................................................................
```

#### 4. 执行失败

核心提示:

**cannot load such file -- /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-66684-11n7slw/MyFastfiles.git/fastlane/tool**

解决：

将 **remote**中的 fastlane 项目 **Fastfile** 中的 **require_relative** 相对导入 xx.rb 代码去掉即可.



## 2. Fastfile 如果引用 ==plugin==

### 1. ==remote== fastlane 项目

#### 1. fastlane/Fastfile

```ruby
lane :my_load_json do |options|
  load_json(json_path: options[:filepath])
end
```

#### 2. fastlane/Pluginfile

```ruby
gem 'fastlane-plugin-load_json'
```

### 2. ==local== fastlane 项目

#### 1. fastlane/Fastfile

```ruby
import_from_git(
  url: 'git@gitee.com:garywade/MyFastfiles.git',
  path: 'fastlane/Fastfile'
)

lane :test do
  UI.success my_load_json(filepath: '/Users/xiongzenghui/Desktop/FastlaneDemo/test.json')
end
```

#### 2. exec local Fastfile ==失败==

```
 ~/Desktop/FastlaneDemo   master ●  bundle exec fastlane test
[✔] 🚀
[01:41:31]: It seems like you wanted to load some plugins, however they couldn't be loaded
[01:41:31]: Please follow the troubleshooting guide: https://docs.fastlane.tools/plugins/plugins-troubleshooting/
[01:41:32]: -----------------------------
[01:41:32]: --- Step: import_from_git ---
[01:41:32]: -----------------------------
[01:41:32]: Cloning remote git repo...
[01:41:32]: $ git clone git@gitee.com:garywade/MyFastfiles.git /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-69782-80afc2/MyFastfiles.git --depth 1 -n
[01:41:32]: ▸ Cloning into '/var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-69782-80afc2/MyFastfiles.git'...
[01:41:32]: $ cd /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-69782-80afc2/MyFastfiles.git && git checkout HEAD fastlane/Fastfile
[01:41:32]: $ cd /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-69782-80afc2/MyFastfiles.git && git checkout HEAD fastlane/actions
[01:41:32]: ▸ error: pathspec 'fastlane/actions' did not match any file(s) known to git
[01:41:32]: Driving the lane 'test' 🚀
[01:41:32]: -----------------------------------------
[01:41:32]: --- Step: Switch to my_load_json lane ---
[01:41:32]: -----------------------------------------
[01:41:32]: Cruising over to lane 'my_load_json' 🚖
+---------------+------+
|     Lane Context     |
+---------------+------+
| PLATFORM_NAME |      |
| LANE_NAME     | test |
+---------------+------+
[01:41:32]: Could not find action, lane or variable 'load_json'. Check out the documentation for more details: https://docs.fastlane.tools/actions

+------+-----------------------------+-------------+
|                 fastlane summary                 |
+------+-----------------------------+-------------+
| Step | Action                      | Time (in s) |
+------+-----------------------------+-------------+
| 1    | import_from_git             | 0           |
| 2    | Switch to my_load_json lane | 0           |
+------+-----------------------------+-------------+

[01:41:32]: fastlane finished with errors

[!] Could not find action, lane or variable 'load_json'. Check out the documentation for more details: https://docs.fastlane.tools/actions
```

- 提示无法找到 **remote Fastfile** 依赖的 **load_json** 这个 **plugin**.
- 因为可能本地，确实 **没装** remote Fastfile 依赖的 **plugin (gem)**

### 3. local fastlane 项目, 必须安装 ==load_json== 这个 plugin

```
 ~/Desktop/FastlaneDemo   master ●  gem install fastlane-plugin-load_json
Successfully installed fastlane-plugin-load_json-0.0.1
Parsing documentation for fastlane-plugin-load_json-0.0.1
Done installing documentation for fastlane-plugin-load_json after 0 seconds
1 gem installed
```

### 6. local fastlane 项目/fastlane/Pluginfile

```ruby
gem 'fastlane-plugin-load_json'
```

### 7. bundle install ==local== fastlane 项目

```
 ~/Desktop/FastlaneDemo   master ●  bundle install
Using CFPropertyList 3.0.0
Using public_suffix 2.0.5
.............................
```

### 8. 再次 exec ==local== fastlane 项目/fastlane/Fastfile (ok)

```
 ~/Desktop/FastlaneDemo   master ●  bundle exec fastlane test
[✔] 🚀
+---------------------------+---------+-----------+
|                  Used plugins                   |
+---------------------------+---------+-----------+
| Plugin                    | Version | Action    |
+---------------------------+---------+-----------+
| fastlane-plugin-load_json | 0.0.1   | load_json |
+---------------------------+---------+-----------+

[01:44:40]: -----------------------------
[01:44:40]: --- Step: import_from_git ---
[01:44:40]: -----------------------------
[01:44:40]: Cloning remote git repo...
[01:44:40]: $ git clone git@gitee.com:garywade/MyFastfiles.git /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-70200-130g2at/MyFastfiles.git --depth 1 -n
[01:44:40]: ▸ Cloning into '/var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-70200-130g2at/MyFastfiles.git'...
[01:44:41]: $ cd /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-70200-130g2at/MyFastfiles.git && git checkout HEAD fastlane/Fastfile
[01:44:41]: $ cd /var/folders/kt/z8c9rz0s5nj68j_d_1bj0y7h0000gn/T/fl_clone20190821-70200-130g2at/MyFastfiles.git && git checkout HEAD fastlane/actions
[01:44:41]: ▸ error: pathspec 'fastlane/actions' did not match any file(s) known to git
[01:44:41]: Driving the lane 'test' 🚀
[01:44:41]: -----------------------------------------
[01:44:41]: --- Step: Switch to my_load_json lane ---
[01:44:41]: -----------------------------------------
[01:44:41]: Cruising over to lane 'my_load_json' 🚖
[01:44:41]: -----------------------
[01:44:41]: --- Step: load_json ---
[01:44:41]: -----------------------
[01:44:41]: Cruising back to lane 'test' 🚘
[01:44:41]: {"name"=>"xiongzenghui", "favor"=>"basketball"}

+------+-----------------------------+-------------+
|                 fastlane summary                 |
+------+-----------------------------+-------------+
| Step | Action                      | Time (in s) |
+------+-----------------------------+-------------+
| 1    | import_from_git             | 0           |
| 2    | Switch to my_load_json lane | 0           |
| 3    | load_json                   | 0           |
+------+-----------------------------+-------------+

[01:44:41]: fastlane.tools finished successfully 🎉
```

这次就能 **成功** 调用 **remote Fastfile** 了.