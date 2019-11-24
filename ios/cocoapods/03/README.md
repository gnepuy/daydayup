[TOC]



## 1、直接使用 cocoapods 管理 pod 存在的问题

- 控制 开发随意添加 各种 pod 组件, 到 App 项目中带来的问题 
- 组件之间的版本依赖关系、App依赖组件版本, 都难以控制
- 如果需要以 development pods 方式, 调试某个 pod 组件时, 还需要去 Podfile 修改 pod 引用方式
- 如果组件实现 源码 和 二进制 切换, 也存在很多的问题
  - 市面上很多切换的实现, 是基于2个仓库, 一个存源码, 一个存二进制
  - 通过环境变量或者什么的, 达到切换, 但是同样需要修改 Podfile 中的 pod 引用方式
  - 二进制失效的问题
- 我们项目中并没有过多使用 podspec repo 来作为组件发布的仓库, 因为这样流程太麻烦, 严重影响内部组件的开发效率



## 2、Podfile 本质

```ruby
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

p self

puts '-' * 50

target 'App' do
  p self
end
```

打印输出

```
╰─± bundle exec pod install
#<Pod::Podfile:0x00007f8d1b1c26e0 @defined_in_file=#<Pathname:/Users/xiongzenghui/Desktop/cocoapods-virus/Examples/App/Podfile>, @internal_hash={}, @root_target_definitions=[#<Pod::Podfile::TargetDefinition label=Pods>], @current_target_definition=#<Pod::Podfile::TargetDefinition label=Pods>>
--------------------------------------------------
#<Pod::Podfile:0x00007f8d1b1c26e0 @defined_in_file=#<Pathname:/Users/xiongzenghui/Desktop/cocoapods-virus/Examples/App/Podfile>, @internal_hash={}, @root_target_definitions=[#<Pod::Podfile::TargetDefinition label=Pods>], @current_target_definition=#<Pod::Podfile::TargetDefinition label=Pods-App>>
Analyzing dependencies
Downloading dependencies
Generating Pods project
Integrating client project
```

可以看到在这个 **Podfile** 文件中的 **self** 是一个 **Pod::Podfile** 类型的 **对象**.

比如添加依赖 `pod 'xxx'` 都是调用的 **Pod::Podfile** 类型的 **对象** 的方法。



## 3、接管 Podfile 中的 pod 'xxx' 过程

```ruby
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

# 1、导入依赖的 gem
require 'virus'

target 'App' do
  Virus::Installer.install(self) # 2、接管 pod 'xxx' 过程
end
```



## 4、