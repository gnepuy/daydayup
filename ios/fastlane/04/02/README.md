[TOC]







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




