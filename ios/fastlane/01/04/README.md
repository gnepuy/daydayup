[TOC]



## 1. before each/all、after each/all、error

```ruby
before_all do |lane, options|
  # ...
end

before_each do |lane, options|
  # ...
end

after_all do |lane, options|
  # ...
end

after_each do |lane, options|
  # ...
end

error do |lane, exception, options|
  if options[:debug]
    puts "Hi :)"
  end
end
```



## 2. 示例

### 1. Fastfile

```ruby
error do |lane, exception, options|
  UI.message("---------------- [error] #{lane} (#{lane.class})")
end

before_each do |lane, options|
  UI.message("---------------- [before_each] #{lane} (#{lane.class})")
end

before_all do |lane, options|
  UI.message("---------------- [before_all] #{lane} (#{lane.class})")
end

after_all do |lane, options|
  UI.message("---------------- [after_all] #{lane} (#{lane.class})")
end

after_each do |lane, options|
  UI.message("---------------- [after_each] #{lane} (#{lane.class})")
end

lane :test1 do
end

lane :test2 do
end

lane :test3 do
  my_action
end

lane :test do
  test1
  test2
  test3
end
```

### 2. bundle exec fastlane test

```
 ~/WORKSPACE/toolbox   master  bundle exec fastlane test
[✔] 🚀

[17:01:07]: ------------------------------
[17:01:07]: --- Step: default_platform ---
[17:01:07]: ------------------------------
[17:01:07]: ------------------------------
[17:01:07]: --- Step: default_platform ---
[17:01:07]: ------------------------------
[17:01:07]: Driving the lane 'test' 🚀
[17:01:07]: ---------------- [before_all] test (Symbol)
[17:01:07]: ---------------- [before_each] test (Symbol)
[17:01:07]: ---------------- [before_each] test1 (Symbol)
[17:01:07]: ----------------------------------
[17:01:07]: --- Step: Switch to test1 lane ---
[17:01:07]: ----------------------------------
[17:01:07]: Cruising over to lane 'test1' 🚖
[17:01:07]: ---------------- [after_each] test1 (Symbol)
[17:01:07]: Cruising back to lane 'test' 🚘
[17:01:07]: ---------------- [before_each] test2 (Symbol)
[17:01:07]: ----------------------------------
[17:01:07]: --- Step: Switch to test2 lane ---
[17:01:07]: ----------------------------------
[17:01:07]: Cruising over to lane 'test2' 🚖
[17:01:07]: ---------------- [after_each] test2 (Symbol)
[17:01:07]: Cruising back to lane 'test' 🚘
[17:01:07]: ---------------- [before_each] test3 (Symbol)
[17:01:07]: ----------------------------------
[17:01:07]: --- Step: Switch to test3 lane ---
[17:01:07]: ----------------------------------
[17:01:07]: Cruising over to lane 'test3' 🚖
[17:01:07]: ----------------
[17:01:07]: --- Step: my ---
[17:01:07]: ----------------
[17:01:07]: my_action: /Users/xiongzenghui/WORKSPACE/toolbox
[17:01:07]: ---------------- [after_each] test3 (Symbol)
[17:01:07]: Cruising back to lane 'test' 🚘
[17:01:07]: ---------------- [after_each] test (Symbol)
[17:01:07]: ---------------- [after_all] test (Symbol)

+------+----------------------+-------------+
|             fastlane summary              |
+------+----------------------+-------------+
| Step | Action               | Time (in s) |
+------+----------------------+-------------+
| 1    | default_platform     | 0           |
| 2    | default_platform     | 0           |
| 3    | Switch to test1 lane | 0           |
| 4    | Switch to test2 lane | 0           |
| 5    | Switch to test3 lane | 0           |
| 6    | my                   | 0           |
+------+----------------------+-------------+

[17:01:07]: fastlane.tools finished successfully 🎉
```

主要关注 **lane , action , plugin** 状态改变 hook 打印信息:

```
[17:01:07]: ---------------- [before_all] test (Symbol)
[17:01:07]: ---------------- [before_each] test (Symbol)
[17:01:07]: ---------------- [before_each] test1 (Symbol)
[17:01:07]: ---------------- [after_each] test1 (Symbol)
[17:01:07]: ---------------- [before_each] test2 (Symbol)
[17:01:07]: ---------------- [after_each] test2 (Symbol)
[17:01:07]: ---------------- [before_each] test3 (Symbol)
[17:01:07]: ---------------- [after_each] test3 (Symbol)
[17:01:07]: ---------------- [after_each] test (Symbol)
[17:01:07]: ---------------- [after_all] test (Symbol)
```

注意：如果你有多个 Fastfile , 则每一个 Fastfile 都可以有 **自己** 的这些 hooks.

