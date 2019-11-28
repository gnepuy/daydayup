[TOC]




## 1. import ==remote== Fastfile

### 1. import from git

```ruby
import_from_git(url: 'https://github.com/fastlane/fastlane')

....
```

### 2. import from git + branch

```ruby
# or
import_from_git(
  url: 'git@github.com:MyAwesomeRepo/MyAwesomeFastlaneStandardSetup.git',
  path: 'fastlane/Fastfile'
)

lane :new_main_lane do
  # 直接调用 MyAwesomeFastlaneStandardSetup 中定义的 lane
end
```

### 3. import from git + branch + version

```ruby
import_from_git(
  url: "git@github.com:fastlane/fastlane.git", # The URL of the repository to import the Fastfile from.
  branch: "HEAD", # The branch to checkout on the repository
  path: "fastlane/Fastfile", # The path of the Fastfile in the repository
  version: "~> 1.0.0" # The version to checkout on the repository. Optimistic match operator can be used to select the latest version within constraints.
)
```

```ruby
import_from_git(
  url: "git@github.com:fastlane/fastlane.git", # The URL of the repository to import the Fastfile from.
  branch: "HEAD", # The branch to checkout on the repository
  path: "fastlane/Fastfile", # The path of the Fastfile in the repository
  version: [">= 1.1.0", "< 2.0.0"] # The version to checkout on the repository. Multiple conditions can be used to select the latest version within constraints.
)
```

基本差不多，只是参数更齐全一点，版本号区间.

### 4. fastlane action `import_from_git` 查看更多参数

```
+---------+-----------------------------------+---------+-------------------+
|                          import_from_git Options                          |
+---------+-----------------------------------+---------+-------------------+
| Key     | Description                       | Env Var | Default           |
+---------+-----------------------------------+---------+-------------------+
| url     | The URL of the repository to      |         |                   |
|         | import the Fastfile from          |         |                   |
| branch  | The branch or tag to check-out    |         | HEAD              |
|         | on the repository                 |         |                   |
| path    | The path of the Fastfile in the   |         | fastlane/Fastfile |
|         | repository                        |         |                   |
| version | The version to checkout on the    |         |                   |
|         | respository. Optimistic match     |         |                   |
|         | operator or multiple conditions   |         |                   |
|         | can be used to select the latest  |         |                   |
|         | version within constraints        |         |                   |
+---------+-----------------------------------+---------+-------------------+
```

https://docs.fastlane.tools/actions/import_from_git/



## 2. import ==remote Fastfile== 示例

### 1. 在本地 iOS App 项目根目录下 fastlane init

```
 ~/Desktop/app   master ●  fastlane init
[✔] 🚀
[✔] Looking for iOS and Android projects in current directory...
[23:37:46]: Created new folder './fastlane'.
[23:37:46]: Detected an iOS/macOS project in the current directory: 'app.xcworkspace'
[23:37:46]: -----------------------------
[23:37:46]: --- Welcome to fastlane 🚀 ---
[23:37:46]: -----------------------------
[23:37:46]: fastlane can help you with all kinds of automation for your mobile app
[23:37:46]: We recommend automating one task first, and then gradually automating more over time
[23:37:46]: What would you like to use fastlane for?
1. 📸  Automate screenshots
2. 👩‍✈️  Automate beta distribution to TestFlight
3. 🚀  Automate App Store distribution
4. 🛠  Manual setup - manually setup your project to automate your tasks
?  4
[23:37:48]: ------------------------------------------------------------
[23:37:48]: --- Setting up fastlane so you can manually configure it ---
[23:37:48]: ------------------------------------------------------------
......................
......................
......................
```

### 2. 编写 app/fastlane/Fastfile 调用 ==remote git repo== 中的 Fastfile 中的 lane

app/fastlane/Fastfile 文件内容:

```ruby
# 仅仅只是 import remote git repo 中的 Fastile 中定义的 lane
# 我们本地的这个 Fastfile 中并【没有】定义任何的 lane
import_from_git(
  url: "git@xxxx.com:xiongzenghui/ios_fastlane.git",
  branch: "master",
  path: "fastlane/Fastfile",
  # version: "~> 1.0.0" # 由于我的 ios_fastlane git 仓库并没有 tag，所以不使用
)
```

### 3. fastlane 直接调用【远程 git 仓库】内的 Fastfile 中定义的 lane

#### 1. fastlane lan1

```
 ~/Desktop/app   master ●  fastlane lan1
[✔] 🚀
...............................................
[23:41:20]: ------------------------------
[23:41:20]: --- Step: default_platform ---
[23:41:20]: ------------------------------
[23:41:20]: ------------------------------
[23:41:20]: --- Step: default_platform ---
[23:41:20]: ------------------------------
[23:41:20]: Driving the lane 'ios lan1' 🚀
[23:41:20]: lan1

+------+------------------+-------------+
|           fastlane summary            |
+------+------------------+-------------+
| Step | Action           | Time (in s) |
+------+------------------+-------------+
| 1    | default_platform | 0           |
| 2    | import_from_git  | 0           |
| 3    | default_platform | 0           |
+------+------------------+-------------+

[23:41:20]: fastlane.tools finished successfully 🎉
```

#### 2. fastlane lan2

```
 ~/Desktop/app   master ●  fastlane lan2
[✔] 🚀
...............................................
[23:42:04]: ------------------------------
[23:42:04]: --- Step: default_platform ---
[23:42:04]: ------------------------------
[23:42:04]: ------------------------------
[23:42:04]: --- Step: default_platform ---
[23:42:04]: ------------------------------
[23:42:04]: Driving the lane 'ios lan2' 🚀
[23:42:04]: lan2

+------+------------------+-------------+
|           fastlane summary            |
+------+------------------+-------------+
| Step | Action           | Time (in s) |
+------+------------------+-------------+
| 1    | default_platform | 0           |
| 2    | import_from_git  | 0           |
| 3    | default_platform | 0           |
+------+------------------+-------------+

[23:42:04]: fastlane.tools finished successfully 🎉
```



## 3. ==action== 只能被 ==local== Fastfile 引用

Add this to the top of your **Fastfile** :

```ruby
# 1、
actions_path '../custom_actions_folder/'

# 2、定义你的各种 lane、ruby 方法、调用上面导入进来的 action
# .......
```

所以应该尽量使用 **plugin** 形式 **替代 action**