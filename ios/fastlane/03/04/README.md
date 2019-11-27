[TOC]



## xcodegen : 读取 yml 生成 xcodeproj

> https://github.com/NovaTecConsulting/fastlane-plugin-xcodegen

```ruby
xcodegen(
  spec: "PATH/project.yml",
  project: "PATH/Project.xcodeproj"
)
```

内部依赖 https://github.com/yonaskolb/XcodeGen 工具.




## ~~altool : 上传 ipa 到 appstore~~

> https://github.com/XCTEQ/fastlane-plugin-altool

```ruby
altool(
  altool_username: ENV["FASTLANE_USER"],
  altool_password: ENV["FASTLANE_PASSWORD"],
  altool_app_type: "ios",
  altool_ipa_path: "./build/Your-ipa.ipa",
  altool_output_format: "xml",
)
```



## clang_analyzer : 代码分析

> https://github.com/SiarheiFedartsou/fastlane-plugin-clang_analyzer

```ruby
clang_analyzer(
  analyzer_path: '~/analyze_tools/bin', # optional
  clean: true, # optional
  workspace: 'Test.xcworkspace', # optional, cannot be used together with `project` option
  project: 'Test.xcodeproj', # optional, cannot be used together with `workspace` option
  configuration: 'Debug', # optional
  sdk: 'iphonesimulator', # optional
  arch: 'i386', # optional
  report_output_path: 'analyzer_report', # optional
  log_file_path: 'analyzer_report.log', # optional
)
```


