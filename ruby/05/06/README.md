[TOC]



## 1. 同一个 gem 可以同时存在 ==多个不同版本== 的包

```
╰─○ ll | grep 'fastlane'
drwxr-xr-x  24 xiongzenghui  staff   768B  3 27  2019 fastlane-2.119.0
drwxr-xr-x  24 xiongzenghui  staff   768B  4 24  2019 fastlane-2.120.0
drwxr-xr-x  24 xiongzenghui  staff   768B  5 15  2019 fastlane-2.122.0
drwxr-xr-x  24 xiongzenghui  staff   768B  6 10 19:45 fastlane-2.125.2
drwxr-xr-x  24 xiongzenghui  staff   768B  6 27 12:04 fastlane-2.126.0
drwxr-xr-x  24 xiongzenghui  staff   768B  8 30 16:41 fastlane-2.129.0
```

可以看到我本机环境，fastlane 同时存在了 6个 不同的版本。

那么问题来了：

- 1) ruby 如何知道我要使用 **哪一个版本** 的 fastlane 包了？
- 2) 假如我删除了其中一些版本, 那么会不会导致我们代码依赖的 fastlane 版本也会自动变化？

确实是有如上这些问题的, 那么 **bundle exec** 就是用来解决这些问题的。



## 2. 通过 bundle exec 固定 gem 包的版本号

### 1. 一个 ruby app 项目

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/App using ‹ruby-2.6.0›
╰─○ tree
.
├── Gemfile
└── main.rb

0 directories, 2 files
```

### 2. 删除本地 gems 目录下所有安装的 fastlane

```
rm fastlane-x.y.z
```

全部删除掉

### 3. App/Gemfile

```ruby
#source "https://rubygems.org"
source "https://gems.ruby-china.com"
git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem 'fastlane', '>= 2.136.0', '< 3.0'
```

### 4. bundle install 安装 fastlane

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/App using ‹ruby-2.6.0›
╰─○ bundle install
Fetching gem metadata from https://gems.ruby-china.com/.........
Fetching gem metadata from https://gems.ruby-china.com/.
Resolving dependencies...
Using CFPropertyList 3.0.1
Using public_suffix 2.0.5
Fetching addressable 2.7.0
Installing addressable 2.7.0
Using atomos 0.1.3
Fetching babosa 1.0.3
Installing babosa 1.0.3
Using bundler 2.0.2
Using claide 1.0.3
Using colored 1.2
Using colored2 3.1.2
Using highline 1.7.10
Using commander-fastlane 4.4.6
Using declarative 0.0.10
Using declarative-option 0.1.0
Using digest-crc 0.4.1
Using unf_ext 0.0.7.6
Using unf 0.1.4
Using domain_name 0.5.20190701
Using dotenv 2.7.5
Using emoji_regex 1.0.1
Fetching excon 0.68.0
Installing excon 0.68.0
Using multipart-post 2.0.0
Fetching faraday 0.17.0
Installing faraday 0.17.0
Using http-cookie 1.0.3
Using faraday-cookie_jar 0.0.6
Using faraday_middleware 0.13.1
Fetching fastimage 2.1.7
Installing fastimage 2.1.7
Using gh_inspector 1.1.3
Using jwt 2.1.0
Fetching memoist 0.16.1
Installing memoist 0.16.1
Fetching multi_json 1.14.1
Installing multi_json 1.14.1
Using os 1.0.1
Fetching signet 0.12.0
Installing signet 0.12.0
Using googleauth 0.6.7
Using httpclient 2.8.3
Fetching mime-types-data 3.2019.1009
Installing mime-types-data 3.2019.1009
Fetching mime-types 3.3
Installing mime-types 3.3
Using uber 0.1.0
Using representable 3.0.4
Using retriable 3.1.2
Using google-api-client 0.23.9
Fetching google-cloud-env 1.3.0
Installing google-cloud-env 1.3.0
Fetching google-cloud-core 1.4.1
Installing google-cloud-core 1.4.1
Using google-cloud-storage 1.16.0
Using json 2.2.0
Using mini_magick 4.9.5
Using multi_xml 0.6.0
Using plist 3.5.0
Fetching rubyzip 1.3.0
Installing rubyzip 1.3.0
Using security 0.1.3
Using naturally 2.2.0
Fetching simctl 1.6.6
Installing simctl 1.6.6
Using slack-notifier 2.3.2
Using terminal-notifier 2.0.0
Using unicode-display_width 1.6.0
Using terminal-table 1.8.0
Using tty-screen 0.7.0
Using tty-cursor 0.7.0
Using tty-spinner 0.9.1
Using word_wrap 1.0.0
Using nanaimo 0.2.6
Fetching xcodeproj 1.13.0
Installing xcodeproj 1.13.0
Using rouge 2.0.7
Using xcpretty 0.3.0
Using xcpretty-travis-formatter 1.0.0
Fetching fastlane 2.136.0
Installing fastlane 2.136.0
Bundle complete! 1 Gemfile dependency, 65 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

### 5. gem install 安装 fastlane 安装另一个不同的版本

```
gem install fastlane -v 2.135.0
```

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/App using ‹ruby-2.6.0›
╰─○ gem install fastlane -v 2.135.0
Fetching fastlane-2.135.0.gem
Successfully installed fastlane-2.135.0
Parsing documentation for fastlane-2.135.0
Installing ri documentation for fastlane-2.135.0
Done installing documentation for fastlane after 14 seconds
1 gem installed
```

### 6. Gems 目录下同时存在2个版本的 fastlane

```
╰─○ ll | grep 'fastlane'
drwxr-xr-x  24 xiongzenghui  staff   768B 11 17 22:56 fastlane-2.135.0
drwxr-xr-x  24 xiongzenghui  staff   768B 11 17 22:55 fastlane-2.136.0
```

### 7. bundle exec main.rb

main.rb

```ruby
system('fastlane --version')
```

bundle exec

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/App using ‹ruby-2.6.0›
╰─○ bundle exec ruby main.rb
fastlane installation at path:
/Users/xiongzenghui/.rvm/gems/ruby-2.6.0/gems/fastlane-2.136.0/bin/fastlane
-----------------------------
[✔] 🚀
fastlane 2.136.0
```

- 可以看到虽然同是存在2个不同版本的 fastlane
- 但是最终 **bundle exec ruby main.rb** 中依赖使用的 fastlane 版本是在 **Gemfile** 中指定的版本
- 而并不是通过 gem install 单独安装的 fastlane 版本

### 8. 修改 Gemfile 使用 2.135.0 版本的 fastlane

```ruby
#source "https://rubygems.org"
source "https://gems.ruby-china.com"
git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem 'fastlane', '2.135.0'
```

### 9. bundle install

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/App using ‹ruby-2.6.0›
╰─○ bundle install
Using CFPropertyList 3.0.1
Using public_suffix 2.0.5
Using addressable 2.7.0
Using atomos 0.1.3
Using babosa 1.0.3
Using bundler 2.0.2
Using claide 1.0.3
Using colored 1.2
Using colored2 3.1.2
Using highline 1.7.10
Using commander-fastlane 4.4.6
Using declarative 0.0.10
Using declarative-option 0.1.0
Using digest-crc 0.4.1
Using unf_ext 0.0.7.6
Using unf 0.1.4
Using domain_name 0.5.20190701
Using dotenv 2.7.5
Using emoji_regex 1.0.1
Using excon 0.68.0
Using multipart-post 2.0.0
Using faraday 0.17.0
Using http-cookie 1.0.3
Using faraday-cookie_jar 0.0.6
Using faraday_middleware 0.13.1
Using fastimage 2.1.7
Using gh_inspector 1.1.3
Using jwt 2.1.0
Using memoist 0.16.1
Using multi_json 1.14.1
Using os 1.0.1
Using signet 0.12.0
Using googleauth 0.6.7
Using httpclient 2.8.3
Using mime-types-data 3.2019.1009
Using mime-types 3.3
Using uber 0.1.0
Using representable 3.0.4
Using retriable 3.1.2
Using google-api-client 0.23.9
Using google-cloud-env 1.3.0
Using google-cloud-core 1.4.1
Using google-cloud-storage 1.16.0
Using json 2.2.0
Using mini_magick 4.9.5
Using multi_xml 0.6.0
Using plist 3.5.0
Using rubyzip 1.3.0
Using security 0.1.3
Using naturally 2.2.0
Using simctl 1.6.6
Using slack-notifier 2.3.2
Using terminal-notifier 2.0.0
Using unicode-display_width 1.6.0
Using terminal-table 1.8.0
Using tty-screen 0.7.0
Using tty-cursor 0.7.0
Using tty-spinner 0.9.1
Using word_wrap 1.0.0
Using nanaimo 0.2.6
Using xcodeproj 1.13.0
Using rouge 2.0.7
Using xcpretty 0.3.0
Using xcpretty-travis-formatter 1.0.0
Using fastlane 2.135.0
Bundle complete! 1 Gemfile dependency, 65 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

直接走的是本地已经安装的 2.135.0 版本的 fastlane.

### 10. bundle exec main.rb

bundle exec

```
╭─xiongzenghui at xiongzenghui的MacBook Pro in ~/Desktop/App using ‹ruby-2.6.0›
╰─○ bundle exec ruby main.rb
fastlane installation at path:
/Users/xiongzenghui/.rvm/gems/ruby-2.6.0/gems/fastlane-2.135.0/bin/fastlane
-----------------------------
[✔] 🚀
fastlane 2.135.0
```

- 此时已经使用了在 Gemfile 中指定的 **fastlane 2.135.0**



## 3. 如果 ==不在 Gemfile 目录或子目录== 下, 执行 bundle exec 会 ==报错==

### 1. 在 Gemfile ==当前== 目录下执行 bundle exec

```
 ~/WORKSPACE/toolbox   master  ll
total 80
-rw-r--r--   1 xiongzenghui  staff   1.1K  7 18 17:26 Gemfile
-rw-r--r--   1 xiongzenghui  staff   7.8K  7 18 18:10 Gemfile.lock
-rw-r--r--   1 xiongzenghui  staff   362B  7 11 14:54 Makefile
-rw-r--r--   1 xiongzenghui  staff   2.9K  7 18 18:20 README.md
drwxr-xr-x  16 xiongzenghui  staff   512B  7 18 17:22 fastlane
-rw-r--r--   1 xiongzenghui  staff   233B  7 17 15:21 jenkins_developer.rb
-rw-r--r--   1 xiongzenghui  staff   189B  7 17 16:08 jenkins_module.rb
-rw-r--r--   1 xiongzenghui  staff   191B  7 11 13:05 jenkins_mr.rb
-rw-r--r--   1 xiongzenghui  staff   192B  7 17 16:08 jenkins_simulator.rb
-rw-r--r--   1 xiongzenghui  staff    13B  7 18 19:16 main.rb
drwxr-xr-x   3 xiongzenghui  staff    96B  7 18 11:04 spec
```

```
 ~/WORKSPACE/toolbox   master  bundle exec ruby main.rb
hello
```

### 2. 在 Gemfile ==子目录== 执行 bundle exec

```
 ~/WORKSPACE/toolbox/spec   master  ll
total 16
-rw-r--r--  1 xiongzenghui  staff   1.8K  7 18 15:05 env
-rw-r--r--  1 xiongzenghui  staff    22B  7 18 19:17 main.rb
```

```
 ~/WORKSPACE/toolbox/spec   master  bundle exec ruby main.rb
spec 子目录
```

### 3. 但是在一个 ==没有 Gemfile== 目录下 执行 bundle exec 会报错

```
 ~/WORKSPACE/non_gemfile  ll
total 8
-rw-r--r--  1 xiongzenghui  staff    19B  7 18 19:20 main.rb
```

```
 ~/WORKSPACE/non_gemfile  bundle exec ruby main.rb
Could not locate Gemfile or .bundle/ directory
```

- 提示找不到 **Gemfile** 或者 `.bundle/ directory` 

- 也就是说只有在 **两种目录** 下, 才能使用 **bundle exec** 执行一些 ruby 代码
  - 1) 包含 **Gemfile** 的 **当前**目录 或者 下面的所有 **子目录**
  - 2) 包含 `.bundle/ directory` 的 **当前**目录 或者 下面的所有 **子目录**

