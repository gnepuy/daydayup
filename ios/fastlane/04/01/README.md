[TOC]



## 1. import ==local== Fastfile

```ruby
# 切记: 文件名 就是【GeneralFastfile】而不是【GeneralFastfile.rb】
import "../GeneralFastfile"

# 这句没事别写，一开始我也不知道，跟着这样写，后来这个 lane 死活不调用
# 这样写之后，表示【重写】"../GeneralFastfile" 文件中的 lane
# 这样写的效果就是【不再调用】"../GeneralFastfile" 文件中的 lane
override_lane :from_general do
  # ...
end
```



## 2. 同时导入 ==多个 Fastfile== 时, 出现 ==同名 lane 冲突==

### 1. FastlaneDemo 项目结构

```
 ~/Desktop/FastlaneDemo   master ●  tree
.
├── Fastfiles
│   ├── OneFastfile
│   └── TwoFastfile
├── Gemfile
├── Gemfile.lock
├── README.en.md
├── README.md
└── fastlane
    ├── Appfile
    ├── Fastfile
    ├── README.md
    └── report.xml

2 directories, 10 files
```

### 2. FastlaneDemo/fastlane/Fastfile

```ruby
import '../Fastfiles/OneFastfile'
import '../Fastfiles/TwoFastfile'

lane :test do
end
```

### 3. FastlaneDemo/Fastfiles/OneFastfile

```ruby
lane :haha do
end
```

### 4. FastlaneDemo/Fastfiles/TwoFastfile

```ruby
lane :haha do
end
```

### 4. exec FastlaneDemo/fastlane/Fastfile ==冲突失败==

```
 ~/Desktop/FastlaneDemo   master ●  bundle exec fastlane test
[✔] 🚀

[!] Lane 'haha' was defined multiple times!
```

**报错** 提示定义了 **同名 lane**



## 3. 上例使用 ==private_lane== 仍然产生 ==冲突==

### 1. FastlaneDemo/Fastfiles/OneFastfile

```ruby
private_lane :haha do
end
```

### 2. FastlaneDemo/Fastfiles/TwoFastfile

```ruby
private_lane :haha do
end
```

### 3. exec FastlaneDemo/fastlane/Fastfile

```
 ~/Desktop/FastlaneDemo   master ●  bundle exec fastlane test
[✔] 🚀

[!] Lane 'haha' was defined multiple times!
```

仍然还是 **报错** 提示定义了 **同名 lane**

### 4. private lane 

- private lane **无法解决** lane 同名冲突
- 仅仅只是标识这个 lane 无法被 **命令行** 直接调用



## 4. ==批量== import Fastfile

### 1. Ruby ==require==

#### 1. 目录结构

```
╰─○ tree
.
├── main.rb
├── tools
│   ├── add.rb
│   ├── mul.rb
│   └── sub.rb
└── tools.rb
```

#### 2. tools.rb

```ruby
Dir[File.expand_path('tools/*.rb', __dir__)].each { |f| require_relative f }
```

#### 3. main.rb

```ruby
require_relative 'tools'
add
sub
mul
```

#### 4. exec main.rb

```
╰─○ ruby main.rb
add ...
sub ...
mul ...
```

### 2. fastlane ==import==

#### 1. 目录结构

```
╰─○ tree
.
└── fastlane
    ├── Fastfile
    └── Fastfiles
        ├── request
        │   ├── login.rb
        │   ├── logout.rb
        │   └── regist.rb
        ├── request.rb
        ├── tools
        │   ├── add.rb
        │   ├── mul.rb
        │   └── sub.rb
        └── tools.rb
```

#### 2. fastlane/Fastfile (顶层 Fastfile)

```ruby
import 'Fastfiles/tools.rb'
import 'Fastfiles/request.rb'
```

#### 3. fastlane/Fastfiles/tools.rb

```ruby
Dir[File.expand_path('tools/*.rb', __dir__)].each { |f| import f }
```

#### 4. fastlane/Fastfiles/request.rb

```ruby
Dir[File.expand_path('request/*.rb', __dir__)].each { |f| import f }
```

#### 4. exec fastlane xx

```
....
```





