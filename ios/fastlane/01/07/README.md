[TOC]



## 1. 转发给 ==ruby 对象==

### 1. Fastfile

```ruby
def prepare(options)
  puts ' 🍏 ' * 10
  p options
end

def doing(options)
  puts ' 🍊 ' * 10
  p options
end

def report(options)
  puts ' 🍐 ' * 10
  p options
end

lane :pipeline_test do |options|
  state = options[:state]
  send(state.to_sym, options)
end
```

### 2. prepare

> `bundle exec fastlane pipeline_test state:'prepare' name:'xiong' age:99`

```
[15:45:41]: Driving the lane 'pipeline_test' 🚀
[15:45:41]:  🍏  🍏  🍏  🍏  🍏  🍏  🍏  🍏  🍏  🍏
{:state=>"prepare", :name=>"xiong", :age=>"99"}
```

### 3. doing

> `bundle exec fastlane pipeline_test state:'doing' name:'xiong' age:99`

```
[15:46:17]: Driving the lane 'pipeline_test' 🚀
[15:46:17]:  🍊  🍊  🍊  🍊  🍊  🍊  🍊  🍊  🍊  🍊
{:state=>"doing", :name=>"xiong", :age=>"99"}
```

### 4. report

> `bundle exec fastlane pipeline_test state:'report' name:'xiong' age:99`

```
[15:47:00]: Driving the lane 'pipeline_test' 🚀
[15:47:00]:  🍐  🍐  🍐  🍐  🍐  🍐  🍐  🍐  🍐  🍐
{:state=>"report", :name=>"xiong", :age=>"99"}
```



## 2. 转发给 ==lane==

### Fastfile

```ruby
lane :pipeline_test_prepare do |options|
  p options[:name]
  p options[:age]
end

lane :pipeline_test_doing do |options|
end

lane :pipeline_test_report do |options|
end

lane :pipeline_test do |options|
  state = options[:state]
  raise '❌ must pass state => state:"prepare"' unless state

  send("pipeline_test_#{state}".to_sym, options)
end
```

### 执行 fastlane

```
bundle exec fastlane pipeline_test state:'prepare' name:'xiong' age:99
bundle exec fastlane pipeline_test state:'doing' name:'xiong' age:99
bundle exec fastlane pipeline_test state:'report' name:'xiong' age:99
```