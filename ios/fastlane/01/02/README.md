[TOC]



## 1. ENV 环境变量读写

```ruby
lane :lan do
  ENV["FASTLANE_PLATFORM_NAME"]
  ENV["FASTLANE_LANE_NAME"]
end
```



## 2. export

### 1. 终端命令行中, export 导出 定义好的 环境变量

```
export FASTLANE_WORKSPACE="<YourApp.xcworkspace>"
export FASTLANE_ITUNESCONNECT_ACCOUNT="<your-itunesconnect-account>"
```

- 为了不和其它常量冲突
- 建议增加一个 **FASTLANE** 前缀

### 2. fastfile 中使用 `ENV['key']` 读取 环境变量

```ruby
lane :appstore do |options|
  puts ENV['WORKSPACE']
  puts ENV['ITUNESCONNECT_ACCOUNT']  
end
```



## 3. dotenv

### 1. 安装 dotenv

```
gem install dotenv
```

### 2. 在 fastlane/ 目录下, 创建 `.env` 文件

```
cd fastlane/
touch .env
```

### 3. 目录结构

```
➜  fastlane git:(master) ✗ tree -a
.
├── .DS_Store
├── .env
├── Fastfile
├── README.md
├── actions
│   ├── git_commit_all.rb
│   └── git_remove_tag.rb
├── report.xml
└── xx

1 directory, 8 files
➜  fastlane git:(master) ✗
```

### 4. 编辑 `.env` 文件

```properties
WORKSPACE=YourApp.xcworkspace
ITUNESCONNECT_ACCOUNT=your-itunesconnect-account
KEY=VALUE
```

### 5. fastfile 中使用 ENV 读取 `.env` 文件

```ruby
lane :appstore do |options|
  puts ENV['WORKSPACE']
  puts ENV['ITUNESCONNECT_ACCOUNT']
end
```

### 6. 执行 fastlane 命令时，通过参数 `--env` 指定读取 `.env` 文件中的参数

```shell
$ fastlane release --env key1
$ fastlane release --env key2
```